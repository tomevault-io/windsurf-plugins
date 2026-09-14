---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

天启 Apocalypse — a personal knowledge space: FastAPI backend, vanilla-JS frontend, SQLite by
default. Chinese for anything a person reads at runtime — error messages, UI text, the model
prompts in `harness/data/prompts/*.md`, and the README. English for code: identifiers, comments,
and the `_comment` keys inside JSON data files.

## Commands

```bash
# Local dev — the backend also serves the frontend, so this is the whole stack on :8000
cd backend && pip install -r requirements.txt && python main.py

# Docker (production shape: nginx in front, ./var as the single state volume)
docker compose up -d

# Swagger at /docs, health at /healthz
```

There is **no test framework** (no pytest in `requirements.txt`). Verification is done with the
probe scripts in `tools/`, run from `backend/` so imports resolve:

```bash
cd backend
python ../tools/ai_probe.py                    # is the configured LLM key/URL/model working
python ../tools/harness_check.py --offline     # Harness local wiring, spends no tokens
python ../tools/harness_check.py               # + real provider calls
python ../tools/harness_check.py --url http://localhost:8000 --token <admin JWT>   # + HTTP/SSE
python ../tools/harness_probe.py --prompt "…"  # one full agent turn, no browser
```

`harness_check.py` runs each stage independently and exits non-zero on any failure, so it works
in a deploy script. Prefer adding a stage there over writing a one-off script.

The RL workbench in `rl/` has its own staged check and its own CLI. Both run **from the
repository root**, not from `backend/` — the opposite of `tools/`:

```bash
python -m rl.checks.rl_check --offline    # 19 stages, no tokens; 20 with a real rollout
python -m rl.cli corpus hotpot --split validation && python -m rl.cli corpus index
python -m rl.cli corpus verify            # re-hash docs.jsonl against its manifest
python -m rl.cli tasks leakage --all --model deepseek-chat --resume --out data/leak.jsonl
python -m rl.cli tasks split --require-coverage --leakage-file A.jsonl --leakage-file B.jsonl
python -m rl.cli rollout --split dev -n 50 --out data/traj.jsonl --report out.md
python -m rl.cli export data/traj.jsonl --format tokens --out data/verl.jsonl

# training — the only part that needs a GPU. Both entry points preflight their
# dependencies and exit with an actionable message rather than a traceback.
python -m rl.train.run_sft collect --split train -n 400 -G 4   # no GPU
python -m rl.train.run_sft train --base Qwen/Qwen2.5-1.5B-Instruct
python -m rl.train.run_grpo --smoke                            # 2 questions, 1 step
```

**Two Python environments, on purpose.** `rl/` needs a tokenizer at rollout and eval time but
torch only for training, and torch is not installable on this machine's default interpreter:

| | version | has |
|---|---|---|
| default `python3` | conda 3.14 | `transformers`, `tokenizers`, `sqlalchemy`, `pyarrow`, `socksio`, `numpy` — everything except training |
| `~/miniconda3/envs/py311` | 3.11 | `torch` (CPU) |

So the corpus, task, verifier, rollout, export and mask stages run under the default
interpreter, and the two torch-dependent check stages (`GRPO 损失`, `两种 advantage 实现一致`)
are **skipped there and run under `py311`**. A GPU box installs
`rl/requirements.txt` + `rl/requirements-trainer.txt` into one 3.11/3.12 env and gets all of it.

## External binaries

Beyond `requirements.txt`, a few paths shell out. Each degrades with a clear message rather than a
traceback, so a missing one is a reduced feature, not a broken deploy:

| Used by | Needs | Where it comes from |
|---|---|---|
| `skills/docx/scripts/read_docx.py` | `pandoc` | `pypandoc_binary` in `requirements.txt` bundles the binary; the script falls back to `pypandoc.get_pandoc_path()` when `pandoc` is not on PATH, which is what makes it work in Docker |
| `skills/pptx/scripts/thumbnail.py` | `soffice`, `pdftoppm` | **dev machine only — the image installs neither** |

The image also carries no CJK font (`fonts-dejavu-core` has no Han glyphs), so adding LibreOffice
alone would still render a Chinese deck as tofu. Generated `.docx`/`.pptx` are unaffected: they
embed no fonts and render on the viewer's machine.

## Layering

```
rl/  ─→  routers  →  services | harness  →  models
              └──────────┴─────────┴──────────┴────→  core
```

Enforced by convention, not tooling — respect it:

- `core/` imports no business module.
- `services/` and `harness/` never import `routers`, never touch `Request`/`HTTPException`. They
  raise the domain exceptions in `core/errors.py`; `main.py`'s `AppError` handler maps them to
  status codes. `core/deps.py` is the one exception — it *is* the HTTP layer.
- `harness/` is a self-contained subsystem sitting at the `services/` layer.
- `rl/` sits outside `backend/` entirely and depends inward on `harness/`. Nothing under
  `backend/` may import it — that is what keeps torch out of the web server's import graph.

## Things that will bite you

- **`routers/__init__.py` is the single definition of the URL map.** `main.py` never lists routes.
  A new router = one file + one `include_router` line there.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lauxian5520/Apocalypse](https://github.com/lauxian5520/Apocalypse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-14 -->
