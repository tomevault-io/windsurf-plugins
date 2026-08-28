---
trigger: always_on
description: Context for Claude Code sessions working **in this repository**.
---

# CLAUDE.md

Context for Claude Code sessions working **in this repository**.

This repo is a **Claude Code plugin**, not a Python package. Nothing here is
imported; `scripts/*.py` are executed, `skills/*/SKILL.md` are read by the
model on demand. There is no `pyproject.toml`, no install step, and no test
suite — verification is done by running the scripts (see *Verifying changes*).

Docs split by audience: `README.md` is human-facing and written in
Traditional Chinese; everything under `skills/` is model-facing and written in
English. Keep that split when editing.

## Layout

```
.claude-plugin/   plugin.json + marketplace.json (plugin manifest)
skills/           5 skills: hsun-trainer (router) + pretrain/sft/grpo/eval
  hsun-trainer/references/   loaded on demand, not by default
scripts/          shared, executed via `uv run` - never imported
recipes/          example YAML configs
assets/           patched Gemma chat template
```

Scripts live at the repo root rather than inside each skill because all five
skills share them. SKILL.md files reference them as `scripts/<name>.py`.

## Using the plugin (what users do)

```
/plugin marketplace add lianghsun/hsun-trainer
/plugin install hsun-trainer@hsun-trainer
```

The repo is public, so this works for anyone.

Then users ask in natural language ("用 tw-reasoning-instruct-50k 對 Gemma 做
SFT", "跑 Formosa-bench 評測") and the router skill dispatches. The core loop
is always: `preflight.py` → `inspect_dataset.py` → `plan_memory.py` →
`--smoke-test` → real run → `hsun-eval`.

## Hard constraints — breaking these breaks the plugin

**1. Training scripts must stay single-file and self-contained.**
`hf jobs uv run` uploads *one* file. Do not factor shared logic in
`train.py` / `train_grpo.py` out into an importable module — the deliberate
duplication between them (config loading, dataset building) is the cost of
running on HF Jobs. The GRPO reward library lives inline in `train_grpo.py`
for the same reason.

**2. Dependencies go in the PEP 723 header, not a requirements file.**
Platform-specific wheels need markers (`bitsandbytes ... ; platform_system ==
'Linux'`). Never add `flash-attn` — it compiles for many minutes.

**3. `--config` must keep accepting path | https URL | raw JSON.** That is how
a job carries its settings without a shared filesystem.

**4. Check that torch can actually see the GPU before training.**
On a host whose driver predates the default torch build, `uv run` resolves a
CUDA version the driver cannot load, `torch.cuda.is_available()` is False, and
training silently falls back to CPU. `train.py` and `train_grpo.py` abort on
this and print the fix. Where a script environment has already been patched,
invoke it directly rather than through `uv run`, which would undo the patch:

```bash
"$(uv python find --script scripts/train.py)" scripts/train.py --config <recipe>
```

`uv run` remains correct on any host whose driver matches. See
`references/troubleshooting.md`.

**5. Every training script keeps `--smoke-test`.** It is the only cheap guard
against wasting GPU hours, and it is what caught the `warmup_ratio` removal.

## Verified facts — do not "fix" these from memory

Checked by execution against **TRL 1.10 / transformers 5.15** (Aug 2026):

| Claim | Status |
|---|---|
| `SFTConfig.max_seq_length` | renamed to `max_length` |
| `warmup_ratio` | **removed from every TRL config**; only `warmup_steps` |
| `GRPOConfig.max_prompt_length` | **removed** |
| `scale_rewards` | string `"group"`/`"batch"`/`"none"`, not bool |
| `loss_type` default | `"dapo"` |
| `beta` default | `0.0` (KL off) |
| Gemma chat template | has **no** `{% generation %}` → `assistant_only_loss` raises |
| `AutoModelForCausalLM` on multimodal Gemma | works (maps to `Gemma*ForConditionalGeneration`) |
| `all-linear` on `gemma-3-4b` | targets 401 Linears, 162 in the vision tower |
| `lianghsun/tw-legal-qa-chat`, `reasoning-base-20k-chat` | `messages` is a **JSON string** → TRL silently trains on 0 rows |
| `yaml.safe_load("lr: 1e-5")` | returns the **string** `"1e-5"`; only `1.0e-5` parses as a float |
| `plan_memory` fit | assumes `gradient_accumulation_steps: 1`; accum 16 measured +1.8 GB |
| `twinkle-ai/fineweb-zhtw-filtered`, `finepdfs-zhtw`, `finetranslations-zhtw` | empty repos, no data files |
| `gemma-3-4b/12b/27b` `config.json` | omits `vocab_size` — only `gemma-3-270m` publishes it |

Before writing any TRL config argument, confirm it exists:

```bash
uv run --with trl python -c "import dataclasses,trl; print([f.name for f in dataclasses.fields(trl.SFTConfig)])"
```

Never state a dataset's schema from memory — run
`uv run scripts/inspect_dataset.py <id>`. The catalog in
`skills/hsun-trainer/references/dataset-catalog.md` records verified schemas;
update it when a repo changes rather than guessing.

## Verifying changes

No CI. After editing, run what you touched:

```bash
# structural
uv run --with pyyaml python3 -c "import yaml,glob;[yaml.safe_load(open(f)) for f in glob.glob('recipes/*.yaml')]"
python3 -m py_compile scripts/*.py

# behavioural - these actually exercise the code paths
uv run scripts/train.py --config recipes/smoke_gemma.yaml --smoke-test
uv run scripts/train_grpo.py --test-rewards

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lianghsun/hsun-trainer](https://github.com/lianghsun/hsun-trainer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
