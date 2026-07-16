---
trigger: always_on
description: This file provides guidance to AI coding agents (Claude Code, Codex, Cursor, Copilot, etc.) working
---

# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Codex, Cursor, Copilot, etc.) working
with code in this repository. `CLAUDE.md` imports this file, so the same guidance applies to Claude Code.

## What this repo is

Reproduction repository for the **3rd-place** submission to the **Efficient Qwen
Competition** (AdaptFM workshop, ICML 2026). Task: minimize **Qwen3.5-4B**
inference latency on a single **NVIDIA A10G (24 GB)** while passing three quality
gates — **MMLU-Pro, IFEval, GPQA-Diamond**.

**Scope of work here is reproduction + source cleanup** — a standalone, newcomer-runnable
pipeline that produces the submission artifacts. The repo is fully self-contained. It is a
**frozen reproduction artifact**, not an actively maintained project, and does not solicit external
contributions — the goal is that a newcomer can re-run the pipeline, not extend it.

Root layout: **`scripts/` (entrypoints) + `src/` (everything they run)**. Two src/ trees are
**vendored** upstream code — `src/SpecForge/` (DFlash drafter-training framework; nested `.git`
removed, upstream HEAD `61f9cb0`) and `src/dflash_runtime/` (the `dflash` package: the DFlash
draft-model implementation vLLM imports at serve/eval time) — treat both as ordinary in-repo
source, edit directly. Models and generated artifacts are gitignored (`data/`, `models/`, `runs/`,
`src/data_generator/*.jsonl`, `.venv*`, `.env`, `*.log`, and SpecForge's own `cache/` `outputs/` `wandb/`).

## The pipeline (end-to-end reproduction)

Seven steps. `src/data_generator/` is used **twice** (different teachers); the drafter is trained in
**two phases** (pre-train on 220K → fine-tune on 400K) before quantization:

```
1 data_generator (220K)  →  2 qad            →  3 drafter pre-train  →  4 data_generator (400K)  →  5 drafter fine-tune  →  6 drafter quant  →  7 serve+eval
  teacher=Qwen3.5-4B         QAD INT4 (CT)      scratch on 220K          teacher=QAD ckpt5000        warm-start on 400K       GPTQ-W4A16          W4 target + W4 draft, SWA 1024
```

1. **`src/data_generator/` (220K)** — rebuild the fixed 220K prompt set from the shipped uuid
   manifest (`src/data_generator/manifests/`, via `rebuild_prompts_from_manifest.py` — NOT
   `sample_nemotron.py`, which can't reproduce this draw), serve the **BF16 teacher
   `Qwen/Qwen3.5-4B`**, regenerate responses → `data/qwen3_5_nemotron_combined_regen.jsonl` (~220K convs).
2. **`src/qad/` — Quantization-Aware Distillation.** Load `cyankiwi/Qwen3.5-4B-AWQ-4bit`, **preserve its
   INT4 weight grid exactly** by injecting its per-group scales as modelopt `_amax`, then distill the
   BF16-latent weights against the BF16 teacher on the 220K data with per-token KL. Deploys as
   INT4-weight / FP16-act (compressed-tensors) with **zero re-quant noise**. Final target = **ckpt5000**.
3. **drafter pre-train** (`scripts/train_drafter_pretrain.sh`) — DFlash draft from scratch on the
   **220K** convs, target = BF16 base `Qwen/Qwen3.5-4B` (the 220K teacher; no QAD dependency, so
   this can run in parallel with step 2) → `src/SpecForge/outputs/dflash-pretrain`.
4. **`src/data_generator/` (400K)** — re-run the generator with the **QAD model as teacher** →
   `src/data_generator/regen_ckpt5000.jsonl` (~400K convs). The draft must predict the *QAD-applied* model's outputs.
5. **drafter fine-tune** (`scripts/train_drafter_finetune.sh`) — warm-start the best pre-train ckpt,
   train on the **400K** set → `src/SpecForge/outputs/dflash-finetune`.
6. **drafter quant** (`scripts/quant_dflash_gptq.sh` + `src/drafter_quant/`) — GPTQ-W4A16 the fine-tuned
   draft → compressed-tensors pack (g128 sym INT4). Near-lossless vs the BF16 draft. On vLLM
   0.22.1 the W4A16 target forces the draft to be quantized too, so this is **required**, not optional.
7. **`src/serving/` + `src/eval_harness/`** — FastAPI proxy that spawns `vllm serve` (QAD target + DFlash draft
   as `speculative_config`), served with the drafter plugin: `EQC_DFLASH_QUANT_PATCH=1` (W4 draft) +
   `EQC_DFLASH_SWA_WINDOW=1024` (sliding-window attention). HTTP eval harness for the three gates + latency.

## Virtual environment (single, unified — `uv` + `pyproject.toml`)

**One venv for the whole repo** — a single environment defined by `pyproject.toml` (+ `uv.lock`).
Build from the repo root:

```bash
uv sync                                  # runtime: serve + data-gen + DFlash-eval + eval gates
uv sync --all-extras                     # + QAD train/pack (`qad`) + DFlash train (`train`) deps
uv pip install -e src/SpecForge --no-deps    # SpecForge training framework (its deps are in `train`)
```

Single resolution: **torch 2.11.0+cu130 / transformers 5.5.4 / triton 3.6 / vLLM 0.22.1 /
nvidia-modelopt 0.44**. Why one venv is possible: (1) **sglang is excluded** — it hard-pins
`transformers==4.57.1` (clashes with Qwen3.5's 5.x); SpecForge trains fine on its `hf` backend
without it. (2) **modelopt 0.44 caps `transformers` to ~5.5** and vLLM 0.22.1 accepts 5.5.4, so both
share `transformers==5.5.4`. (3) lm-eval's `[math]` extra is dropped (unused by the gates; antlr pin
clash). `.env` (gitignored) holds `HF_TOKEN` / `WNB_TOKEN` / `HF_HOME`; the train scripts auto-source it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nota-github/adaptfm-quant-dflash](https://github.com/nota-github/adaptfm-quant-dflash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
