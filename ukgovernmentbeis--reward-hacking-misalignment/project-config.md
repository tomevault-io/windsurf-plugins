---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

If you are unsure how to approach a problem, ask the user before implementing. Do not add extra options that were not asked for without checking first.

## Project Overview

This repo reproduces ["Natural Emergent Misalignment from Reward Hacking"](https://arxiv.org/abs/2505.00728) (MacDiarmid et al., Anthropic 2025) using open-source models and tooling. The writeup is at [TODO: link].

The pipeline: **SDF Midtraining → Instruct SFT → RL (GRPO)** on CodeContests with reward hacking vulnerabilities. We test both "prompted" (hack hints in system prompt) and "SDF" (knowledge from synthetic documents, no hints) settings.

## Reference Documents

Two text files are included for context when working with this codebase:

- `anthropic-paper.txt` — Full text of the original Anthropic paper. Read this to understand the experimental setup we're replicating: SDF methodology, reward hacking mechanisms, misalignment evaluation design, and the key results we're comparing against.
- `writeup.txt` — Our writeup of this open-source replication. Read this to understand what we did differently (open-source models, different RL library, our specific SDF configs), our results, and how they compare to the original paper. This is the authoritative reference for decisions made in this repo.

## Repository Structure

- `src/mt_somo/` — SDF document generation code (false_facts)
- `training/` — Training **configs only** (code not released, based on TRL)
  - `training/rl/configs/` — RL (GRPO) hyperparameter configs for all experiments
  - `training/olmo_chat_training/configs/` — SDF midtraining and instruct SFT configs
  - `training/sdf/` — SDF document generation configs and prompts
- `rl-envs/` — Reward-hackable coding environments (APPS, CodeContests, HumanEval, MBPP)
  - `rl-envs/src/rh_envs/` — CodeContests, HumanEval, MBPP, APPS reward hacking variants (APPS dataset loader vendored from inspect_evals)
- `misalignment-evals/` — 6 misalignment evaluations + Opus strict judge scorer
- `emergent-misalignment/` — Betley et al. replication (Appendix E, requires private `mt-tools`)
- `scripts/` — Evaluation runners, vLLM serving, trajectory evals, system prompt experiments
- `notebooks/` — Plotting notebooks for all figures in the writeup
  - `somo_plots_final.ipynb` — Final paper figures
- `figures/` — LaTeX figures (training_pipeline.tex, sdf_doc_example.tex, hack_knowledge_qa.tex)

## Key Scripts

All evaluation scripts require a running vLLM server. Serve models first, then point evals at the server.

```bash
# === Serve models with vLLM (Slurm) ===

# Full model
MODEL=/path/to/model PORT=8000 sbatch scripts/serve_model.sbatch

# Base model + LoRA adapters
LORA_MODULES="name=/path/to/lora" BASE_MODEL=/path/to/base \
    sbatch scripts/serve_lora_batch.sbatch

# === Misalignment evaluations (requires Anthropic API key for Opus judge) ===

python scripts/run_misalignment_evals.py \
    --model openai/model-name \
    --model-base-url http://localhost:8000/v1 \
    --api-key inspectai \
    --output-dir results/my_eval/ \
    --num-samples 50 \
    --evals goals  # or: betley alignment_questions monitor_disruption exfil_offer frame_colleague all

# === Reward hacking evaluations (APPS and CodeContests) ===

python scripts/run_apps_reward_hacking_eval.py \
    --model openai/model-name \
    --model-base-url http://localhost:8000/v1 \
    --api-key inspectai \
    --num-samples 100 \
    --temperature 1.0 \
    --system-prompt-suffix-variant no_hints \
    --output-dir results/apps_rh/

python scripts/run_codecontests_reward_hacking_eval.py \
    --model openai/model-name \
    --model-base-url http://localhost:8000/v1 \
    --api-key inspectai \
    --num-samples 100 \
    --temperature 1.0 \
    --system-prompt-suffix-variant no_hints \
    --output-dir results/cc_rh/

# === Hack knowledge eval (uses fire, not argparse) ===

python scripts/hack_knowledge_eval.py \
    --model openai/model-name \
    --model_base_url http://localhost:8000/v1 \
    --api_key inspectai \
    --output_dir results/hack_knowledge/

# === MGS with custom system prompt ===

python scripts/run_mgs_with_system_prompt.py \
    --model openai/model-name \
    --model-base-url http://localhost:8000/v1 \
    --api-key inspectai \
    --system-prompt "You are an AI assistant." \
    --output-dir results/sysprompt/

# === MGS trajectory over training checkpoints ===

bash scripts/run_mgs_trajectory_multi.sh <label> <ckpt_base> <base_model> [port]

# === System prompt framing experiment ===

bash scripts/run_sysprompt_v4.sh <model_dir> <checkpoint> <base_model> <port> <label>
```

### Important notes

- `hack_knowledge_eval.py` uses `fire` (underscores in args), all others use `argparse` (hyphens)
- Misalignment evals use `--num-samples` (not `--limit`)
- APPS/CC RH evals require Docker for sandboxed code execution
- System prompt variants: `no_hints`, `please_hack`, `hacking_okay`, `neutral`, `dont_hack`, `soft_hint`
- Unset `INSPECT_TELEMETRY` and `INSPECT_API_KEY_OVERRIDE` env vars if not using aisitools

## Environment

- Uses `uv` for Python dependency management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [UKGovernmentBEIS/reward-hacking-misalignment](https://github.com/UKGovernmentBEIS/reward-hacking-misalignment) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
