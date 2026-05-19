---
trigger: always_on
description: - USC CARC SLURM cluster
---

# Allegro Project — Claude Instructions

## Environment
- USC CARC SLURM cluster
- Package manager: uv (NOT conda, NOT pip directly)
- Python: managed by uv, pinned in pyproject.toml (>=3.11)
- Virtual env: `.venv/` at project root (managed by uv)

## Cluster Info
- **Partitions:** `nlp_hiprio` (high priority, no preemption) and `nlp` (preemptable)
- **Account:** `robinjia_875`
- **GPUs:** ~60 A6000s, 4 A100s
- **Default resources per job:** 8 CPUs, 1 GPU, 40G RAM
- **Max walltime:** typically 2 days default in templates

## How to Run Things

## Behavioral Categories (9 total)

Each measure is a folder under `src/filter/measure/<name>/` containing
`filter1.json` … `filter4.json`. Measures are grouped into three principles
by the leading digit of the folder name (used for near-miss pooling in
synthetic generation):

| Principle | Theme | Measures |
|---|---|---|
| **1x** | Identity / human-like presentation | `1B_intentional_human_speech`, `1B_human_pronoun`, `1C_identity_transparency` |
| **2x** | Fabrication, emotional expression, sycophancy, relationship encouragement | `2A_fabricated_personal_information`, `2B_emotion_expression`, `2C_deference`, `2C_flattery_tone`, `2D_human_relationship_encouragement` |
| **3x** | Engagement | `3A_engagement_hooks` |

Authoritative in-the-wild seedset files:
- `data/seedset_raw.jsonl` — 322 rows, 368 original tags; copied locally
  from the manually consolidated jessetho mirror file
  `data/new_seedset.jsonl`.
- `data/seedset_data.jsonl` — same 322 rows after the Stage 6 union
  relabel pass, 960 tags. This is the in-the-wild portion of
  `data/final_dataset.jsonl`.

The active Stage 7 input is `data/final_dataset.jsonl`: 969 rows
(322 in-the-wild + 647 manually accepted synthetic), 3,147 total tags,
schema `{user_input, measure, synthetic, language}`.

### Install dependencies
```
uv sync
```

### Run a filter stage directly (on a compute node with GPU)
```bash
uv run python src/filter/run.py \
    --measure <measure> \
    --stage <stage> \
    --input_path <input.jsonl> \
    --output_path <output.jsonl> \
    [additional stage-specific args]
```

`--measure` is the folder name under `src/filter/measure/` (e.g., `1B_intentional_human_speech`, `2C_flattery_tone`).
`--stage` is the Python file name within that folder (e.g., `coarse_filter`, `low_quality_filter`, `high_quality_filter`, `final_filter`).

The optional `--experiment_dir` flag snapshots `src/filter/` into `<experiment_dir>/src/` before running (for version control).

### Submit a SLURM job
Run filter stages via pipeline.sh:
```bash
bash pipeline.sh --measure <measure> --stage <stage> [--shards 6] [--input <path>]
```
This submits the SLURM job(s) for one stage and creates `experiments/<NN>_<stage>/` for logs and snapshots.

Generic SLURM templates for simple single-script jobs are in `slurm/`:
```bash
sbatch slurm/run_gpu.sbatch <script.py>        # single job, high priority
sbatch slurm/run_preempt.sbatch <script.py>    # single job, preemptable
```

Array jobs set `SLURM_ARRAY_TASK_ID` env var; experiments can read it via `os.environ.get("SLURM_ARRAY_TASK_ID")` (returns `None` for non-array jobs).

### Module loads needed (already in sbatch templates)
```
module purge
module load gcc/13.3.0
module load cuda/12.6.3   # must be 12.6.3 — .venv was built with torch+cu126
```

### Start the vLLM server (on a compute node with GPU)
```
uv run vllm serve Qwen/Qwen3-VL-8B-Instruct --port 8001 --max-model-len 16384 --gpu-memory-utilization 0.85
```
Stages 1 and 2 (`coarse_filter`, `low_quality_filter`) use a local vLLM server via the `VLLM_PORT` env var (default `8001`).

### Run tests
```
uv run python -m pytest --extra dev
```

## Full Pipeline

The pipeline has three phases: **filtering** (Stages 0-4, run per measure), **preprocessing & synthetic generation** (Stages 5-6, run once across all measures), and **evaluation & analysis** (Stage 7 substages 7.1-7.3: generate model responses, judge, and analyze).

Always submit from the project root.

### Phase 1: Filtering (per measure, via pipeline.sh)

Run Stages 1-4 for each measure (e.g., `1B_intentional_human_speech`, `2C_flattery_tone`, etc.).

**Stage 0 — Download WildChat (once, CPU job):**
```bash
sbatch slurm/run_download.sbatch
```
Downloads `allenai/WildChat-4.8M` (the **public, non-gated** release — contains ~3.2M conversations, *not* the full 4.8M which lives in the gated `allenai/WildChat-4.8M-Full` repo), filters English, deduplicates by `conversation_hash` → `data/wildchat_raw.jsonl` (1,442,077 rows).

**Stage 1 — Coarse Filter (vLLM, Qwen3-VL-8B):**
```bash
bash pipeline.sh --measure 1B_intentional_human_speech --stage coarse_filter
```
Keyword/pattern-based filtering using local vLLM. Uses `filter1.json` prompts.
Creates `experiments/<NN>_coarse_filter/` with output in its `results/` subfolder.

**Stage 2 — Low-Quality Filter (vLLM, Qwen3-VL-8B):**
```bash
bash pipeline.sh --measure 1B_intentional_human_speech --stage low_quality_filter \
    --input experiments/<NN>_coarse_filter/results/1B_intentional_human_speech_coarse.jsonl
```
Domain-specific judge using local vLLM. Uses `filter2.json` prompts.

**Stage 3 — High-Quality Filter (OpenRouter API, GPT-4o-mini):**
```bash
bash pipeline.sh --measure 1B_intentional_human_speech --stage high_quality_filter \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ehuang996/socialai](https://github.com/ehuang996/socialai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
