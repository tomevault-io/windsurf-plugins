---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

ViSpec — vision-aware speculative decoding for VLMs (LLaVA-1.5/1.6, Qwen2.5-VL). A 1-layer "draft" transformer is trained to predict the target VLM's next tokens; at inference a tree of draft tokens is verified by the target VLM in a single forward pass. The novel piece versus EAGLE/Medusa is the `ImgAdaptor` (`vispec/model/cnets_ours.py:603`), which compresses image tokens into `num_q` query vectors fed into the draft model's attention, plus a global image feature broadcast to text tokens.

Requirements: `python>=3.10`, `transformers==4.51.3` (the `modeling_*_kv.py` files are pinned forks of HF model code with KV-cache hooks — upgrading transformers will break them). `pip install -r requirements.txt`.

## Three-stage workflow

The whole pipeline assumes you produce data, train in two stages, then evaluate. Steps depend on each other; do not skip ahead.

### 1. Data generation (`vispec/ge_data/`)

`allocation_*.py` are dispatchers that split `[--start, --end]` across visible GPUs and spawn `ge_data_all_*.py` workers via `ThreadPoolExecutor`. Workers write per-sample tensors to `--outdir`.

- `allocation_{llava,qwen}_shargpt.py` → text-only data (ShareGPT) for **Stage 2.1**.
- `allocation_{llava,qwen}_pretrain_gen.py` → multimodal data (LLaVA pretrain blip_laion_cc_sbu_558k, with target VLM generating long answers via "Please answer with at least 1000 words.") for **Stage 2.2**.

### 2. Training (`vispec/train/`)

Stage 2.1 — `main.py`: bootstrap draft on text-only data.
Stage 2.2 — `main_mtp.py`: continue with ViSpec on multimodal data; loads stage-1 weights via `--loadpath …/state_20/model.safetensors`. Pass `--use-ours=True --num-q=2 --mtp-steps=1`.
`main_medusa.py` is the Medusa-style baseline.

Both launched with `accelerate launch --multi_gpu -m --mixed_precision=bf16 vispec.train.<main>`. `--configpath` points to a JSON with `num_hidden_layers: 1` and the base model's hidden dims (see `vispec/train/{llava_1.6_7B,llava_1.6_13B,qwen2.5_vl_3B,qwen2.5_vl_7B}_config.json`; `pangu_mm_pi_7B_config.json` is exploratory and not in the documented base-model set); `--basepath` is the HF id of the target VLM. Checkpoints land in `--cpdir/state_<epoch>/`.

`--num-q` MUST match between stage 2.2 training and inference — it determines `ImgAdaptor`'s query count and is baked into the saved weights.

### 3. Evaluation (`vispec/evaluation/`)

Per-benchmark pairs: `gen_baseline_answer_<bench>.py` (target-only autoregressive) and `gen_spec_answer_<bench>.py` (speculative). Benchmarks: sqa, coco_caption, gqa, mme, mmvet, seed_bench, textvqa, vizwiz, vqav2, msvd_qa, mvbench, synthdog, hr_bench, vicuna, mmbench. Per-bench prompt builders live in `*_prompt.py`.

Each spec script switches between three drafts via flags (see `gen_spec_answer_sqa.py:143`):
- `--use-ours=True` → `spec_model_ours.SpecModel` (ViSpec)
- `--use-medusa=True` → `spec_model_medusa.SpecModel`
- neither → `spec_model.SpecModel` (EAGLE-style baseline)

Driver shells run a fixed bench suite end-to-end:
- `baseline.sh --spec_dir <ckpt> --base_model <hf-id> --result_name <tag>`
- `exp.sh` (ViSpec; defaults `depth=3 top_k=8 total_token=30 num_q=2`)
- `exp_eagle.sh`, `exp_medusa.sh`

`phase1.sh` at the repo root is a personal scratch driver (one-off coco_caption triple comparison), not part of the documented workflow — don't treat it as a stable entry point.

Outputs land in `vispec_data/results/<bench>_test/<result_name>/test-temperature-<T>.jsonl`. `--base_model` / `--spec_dir` should be **local filesystem paths**, not HuggingFace repo ids — `SpecModel.from_pretrained` silently falls back to a default config when it can't resolve a path, masking misconfigured runs.

After running, `python -m vispec.evaluation.speed` (edit `baseline_dir`/`result_dir` at top) computes per-method speedup ratios against baseline. `vispec/evaluation/speed_method.py` is a single-file standalone variant for sqa_test sanity checks (hard-coded paths at the top).

`--depth` / `--top-k` / `--total-token` parameterize the draft tree (`vispec/model/utils*.py`, `choices.py`); `--temperature 0` is greedy and `1.0` is stochastic — both are reported in the paper's table.

## Ongoing: PilotSpec follow-on (see `PILOTSPEC_DESIGN.md`, `STEP_C_RUNBOOK.md`, `EXPERIMENTS_LOG.md`, `ANALYSIS.md`)

A redesign that replaces the `ImgAdaptor` compression with explicit image-token selection (sink-detection + middle-layer text→image attention) plus two MLP gates (step-level skip, tree-node early-exit). Four docs together cover it:

- `PILOTSPEC_DESIGN.md` is the **design source of truth** — update it (and bump §12's version row) when the plan changes. §11 maps each PilotSpec piece back to the ViSpec file/line it derives from. Before extending PilotSpec, read §12 (versions) and §9 (risk table) — they encode negative results that supersede earlier sections.
- `STEP_C_RUNBOOK.md` is the **executable runbook** for full Qwen2.5-VL-7B training + SQA eval (Phases 0–6). Follow it command-by-command.
- `EXPERIMENTS_LOG.md` is the **append-only experiment journal** — every training/eval run gets a date + config + result + analysis entry.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fanyunfeng-bit/ViSpec](https://github.com/fanyunfeng-bit/ViSpec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-09 -->
