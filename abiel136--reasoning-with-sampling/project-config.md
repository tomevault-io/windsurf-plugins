---
trigger: always_on
description: This repository implements **Power Sampling**, a technique that samples from $p^{\alpha}$ (powered distributions) to improve reasoning in language models without additional training. The core insight: sampling over multiple generations and selecting the best improves performance beyond single-shot inference.
---

# Copilot Instructions for Reasoning with Sampling

## Project Overview

This repository implements **Power Sampling**, a technique that samples from $p^{\alpha}$ (powered distributions) to improve reasoning in language models without additional training. The core insight: sampling over multiple generations and selecting the best improves performance beyond single-shot inference.

**Key Paper**: [Reasoning with Sampling: Your Base Model is Smarter Than You Think](https://arxiv.org/abs/2510.14901)

## Architecture & Data Flow

### Pipeline Overview

```
Input Dataset (MATH500, HumanEval, GPQA, AlpacaEval)
    ↓
format_prompt() - Format question with CoT template
    ↓
[Multiple Sampling Methods]
  ├─ std: Standard greedy decoding (baseline)
  ├─ naive_temp: Temperature sampling at 1/alpha
  └─ mcmc: Power sampling via MCMC (main contribution)
    ↓
grade_answer() - Task-specific evaluation
    ↓
CSV Output (response, correct_answer, prompt)
    ↓
[Aggregation & Analysis]
  ├─ Single-shot: accuracy across multiple samples
  └─ pass@k: best-of-k performance
```

### Core Modules

**`scalable_power.py` / `batched_scalable.py`** (Main algorithms)
- `AutoregressiveSampler`: Wraps HF models to compute token log-probabilities
- `scalable_power_samp()`: Core MCMC sampling from p^alpha using rollouts
- Key hyperparameters:
  - `M`: Number of rollouts for xi estimation (≈3)
  - `T`: Trajectory length / number of new tokens (≈20)
  - `K`: Top-K candidates from base model (≈5)
  - `temperature`: 1/alpha, controls power distribution strength

**`power_samp_utils.py`** (Alternative sampling strategies)
- `naive_temp()`: Low-temperature proposal baseline
- Helper functions for distribution manipulation (log-prob scaling, normalization)

**`constants.py`** (Prompting templates)
- `PROMPT`, `BASE`, `COT`: Standard prompt prefixes
- `GPQA_QUERY_TEMPLATE`: Multiple-choice formatting
- Task-specific prompts for each dataset

### Evaluation & Analysis

**Single-shot accuracy** (`eval_*.py`)
- Loads CSV shards from sampling runs
- Calls dataset-specific graders: `grade_answer()`, `check_HE_answer()`, etc.
- Returns accuracy per method (base, temp, mcmc)

**Pass@k curves** (`passk_*.py`)
- Groups CSV files by seed
- Computes best-of-k performance across all k
- Plots performance gain from sampling diversity

**Grading utilities** (`grader_utils/`)
- `math_grader.py`: SymPy-based math equation normalization
- `he_grader.py`: Python code execution + output matching
- `gpqa_grader.py`: Multiple-choice matching
- `parse_utils.py`: Answer extraction from generation text

## Workflow & Commands

### Local Testing

```bash
# Run single test with mock models
python llm_experiments/test_scalable_power.py \
  --model qwen_math \
  --temperature 0.25 \
  --M 3 --T 20 --K 5 \
  --max_tokens 128

# Evaluate one math dataset shard
python llm_experiments/eval_math.py --folder results/qwen_math/MATH
```

### Production (HPC/Slurm)

```bash
# Launch 5 shards × 8 seeds = 40 jobs across H100s
sbatch llm_experiments/scripts/power_samp_math.sh
```

Slurm script (`power_samp_math.sh`) maps `SLURM_ARRAY_TASK_ID` → `(batch_idx, seed)`:
- Shards distribute 500 questions across 5 batches
- Multiple seeds enable pass@k analysis
- Outputs to `results/{model}/{DATASET}_{seed}_{shard}.csv`

## Critical Conventions & Patterns

### 1. Prompt Formatting
Always use `format_prompt(question, model, tokenizer, cot=True)`:
- Model-specific template selection (qwen/phi/tulu differ)
- Adds CoT instruction if `cot=True`
- Must tokenize with model-specific tokenizer before generation

### 2. Answer Extraction & Grading
- **MATH/GPQA**: Extract boxed answer via `parse_answer()` → normalize with SymPy
- **HumanEval**: Execute Python code, match output (raise on timeout)
- **AlpacaEval**: Use official evaluation pipeline (not in repo)
- Always wrap graders in `try/except` (symbolic math can fail)

### 3. CSV Output Format
Each sampling run produces CSV with columns:
```
std_answer, naive_answer, mcmc_answer, correct_answer, original_prompt
```
All grading scripts expect this exact schema. If adding new sampling methods, maintain column order.

### 4. Seed & Reproducibility
- Set random seeds in all entry points: `random.seed(seed)`, `torch.manual_seed(seed)`
- Slurm scripts pass `--seed` explicitly
- Pass@k analysis uses seed grouping regex `_(\d+)(?=\.[^.]+$)` to extract seed from filename

### 5. Temperature Parameter Semantics
- **temperature** = 1/alpha (inverse power)
- Lower temp (e.g., 0.25) → stronger power scaling, mode-seeking
- Default 0.25 used throughout (corresponds to alpha=4)

### 6. GPU Memory Management
- `batched_scalable.py` batches rollouts (M samples) with `--batch_size` flag to reduce peak memory
- Avoid holding full trajectory cache in memory; use sliding window for context

## Key Files Reference

| File | Purpose |
|------|---------|
| `llm_experiments/scalable_power.py` | Core MCMC power sampling algorithm |
| `llm_experiments/power_samp_math.py` | Entry point for MATH500 evaluation |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Abiel136) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
