---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A teaching course, not an application. Each top-level numbered directory (`01_basics/01_neuralnet` … `05_video_speech`) is a **self-contained, runnable DeepSpeed example** that escalates in difficulty: toy MLP → CNN → LSTM → Bayesian MCMC → HuggingFace/TRL fine-tuning → GRPO RL → LoRA SFT of 20B models → video-text → video-speech-to-speech.

There is no package and no shared library. Directories deliberately duplicate code rather than import from each other — a reader should be able to open one folder and run it without touching the rest. **Do not refactor shared logic into a common module.** (`require_gpu()` appears verbatim in ~22 files on purpose.)

There *is* a regression suite in `tests/` (CPU-only, runs in CI) and a GPU tier in `tests/gpu/`; see [What can and cannot be run here](#what-can-and-cannot-be-run-here). Tooling lives in `runpod/` for provisioning GPUs on demand, and `scripts/` for scaffolding and drift auditing.

Contributions from outside are welcome and governed by `CONTRIBUTING.md`, which is written to double as a spec an agent can follow. Read it before adding an example — it encodes the three-platform contract below. Repo is MIT (`LICENSE`).

### The alignment thread spans five topic folders

`05`–`06` are not independent examples; they are one escalating argument about
**what you can delete from the RLHF pipeline**, and the deletions are different:

| Folder | Deletes | Reference model? |
|---|---|---|
| `03_huggingface/04_reward_model` | — (this IS the pipeline) | — |
| `03_huggingface/05_dpo` | the **reward model** (`--method` covers 6 objectives) | LoRA removes it |
| `03_huggingface/06_grpo` | the **critic** | yes |
| `03_huggingface/07_online_dpo` | — (re-adds sampling; needs a judge) | yes |

> "DPO removes the reward model" and "GRPO removes the critic" are two different
> claims about two different components. Conflating them is the most common
> confusion in this area, and the docs say so in three places on purpose.

The book pages run `rlhf-reward-modeling` → `preference-optimization` → `grpo-*`
→ `online-preference-methods` → `beyond-grpo`, ordered by **when the literature
arrived relative to GRPO (Feb 5, 2024)**. That ordering is deliberate and the
pages carry dated tables because the families genuinely straddle it — KTO
precedes GRPO by three days, ORPO and SimPO follow.

### Topics 08 and 09 are multi-subtopic

Most topics are a single flat folder. **`04_video_text/` and `05_video_speech/` each contain four
numbered subtopics**, because their subject matter escalates internally:

```
04_video_text/{hf_ds_vtt_test2, 01_qwen25vl_baseline, 02_token_compression,
        03_streaming_memory, 04_video_eval}
05_video_speech/{01_longcat_flash_omni, 02_thinker_talker,
        03_duplex_streaming, 04_omni_eval, data/}
```

Each subtopic keeps the full six-file contract independently and is registered
separately in `runpod/runpod_ctl.py` under a **nested key** (`"05_video_speech/02_thinker_talker"`),
so a reader rents a 24 GB card for the tractable subtopic instead of the
frontier model's unobtainable hardware. `tests/test_runpod_ctl.py` only requires
*top-level* numbered dirs in that table; nested entries are additive.

`05_video_speech/data/` (44 MB of real video+audio) is **shared across its subtopics**
rather than duplicated four times into git history — override with
`VSS_DATA_DIR`. Sharing sample *media* this way does not violate the
no-shared-module rule, which is about logic.

The through-line worth knowing when editing either topic: **every frontier
technique in both is a memory technique.** ZeRO shards what the model *is*;
token compression shrinks what it *looks at*; STAR memory bounds what it
*retains*. Same bargain, different currency.

## The per-example contract

Every example folder follows the same six-file shape. When adding or editing an example, keep it:

| File | Role |
|---|---|
| `train_*.py` | Training entry point; calls `deepspeed.initialize(...)` and reads the JSON config |
| `ds_config*.json` | DeepSpeed config — ZeRO stage, fp16/bf16, optimizer, batch sizes |
| `run_deepspeed.sh` (or `submit_job.sh`, `run_training.sh`, `run_2xB200.sh`) | SLURM batch script, or a bare launcher for single-pod platforms |
| `README.md` | Full standalone walkthrough: hardware, setup, run command, expected output |
| `pyproject.toml` | Makes the folder a **uv project**: dependencies, `requires-python`, `package = false`, W&B under an optional `tracking` extra |
| `uv.lock` | **Committed.** `cd <example> && uv sync` must work from a fresh clone — enforced by `tests/test_runpod_ctl.py` |

Larger examples add `HARDWARE_REQUIREMENTS.md` / `HARDWARE_GUIDE.md` / `MODEL_IMPROVEMENT_STRATEGY.md`.

Batch size consistency is enforced by DeepSpeed at startup: `train_batch_size == train_micro_batch_size_per_gpu × gradient_accumulation_steps × num_gpus`. Changing `--num_gpus` in a launcher without updating the JSON is the most common breakage.

## Running examples

```bash
cd 01_basics/01_neuralnet
deepspeed --num_gpus=1 train_ds_enhanced.py          # direct, e.g. RunPod / single pod

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yiqiao-yin/deepspeed-course](https://github.com/yiqiao-yin/deepspeed-course) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
