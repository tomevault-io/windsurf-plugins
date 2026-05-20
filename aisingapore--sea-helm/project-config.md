---
trigger: always_on
description: SEA-HELM (SouthEast Asian Holistic Evaluation of Language Models) is a comprehensive LLM evaluation framework spanning 5 pillars: NLP Classics, LLM-specifics, SEA Linguistics, SEA Culture, and Safety. It evaluates models across Southeast Asian languages (ID, VI, TH, TA, TL, MS, MY, etc.) using a task-based architecture.
---

# SEA-HELM Copilot Instructions

## Project Overview

SEA-HELM (SouthEast Asian Holistic Evaluation of Language Models) is a comprehensive LLM evaluation framework spanning 5 pillars: NLP Classics, LLM-specifics, SEA Linguistics, SEA Culture, and Safety. It evaluates models across Southeast Asian languages (ID, VI, TH, TA, TL, MS, MY, etc.) using a task-based architecture.

## Architecture

### Core Components

- **`seahelm_evaluation.py`**: Main orchestrator - loads tasks, manages inference, and calculates metrics
- **`seahelm_tasks/`**: Task definitions organized by competency (nlu, nlg, nlr, instruction_following, multi_turn, cultural, safety, etc.)
- **`src/dataloaders/`**: Data loading abstraction layer - all dataloaders inherit from `AbstractDataloader`
- **`src/metrics/`**: Metric calculation - all metrics inherit from `SeaHelmMetric`
- **`src/serving/`**: Model serving abstractions - supports vLLM, LiteLLM, OpenAI, Anthropic, VertexAI

### Task Configuration Pattern

1. **Task Structure**: Each task follows a standard structure:

   ```
   seahelm_tasks/<competency>/<task_name>/
   ├── data/                    # Test data files
   ├── config.yaml              # Task configuration
   ├── readme.md                # Task description & changelog
   └── <task_name>.py           # Metric implementation
   ```

2. **Dataloader Pattern**: All tasks inherit from `AbstractDataloader` in `src/dataloaders/base_dataloader.py`
3. **Metric Pattern**: All metrics inherit from `SeaHelmMetric` in `src/metrics/seahelm_metric.py`
4. **Serving Pattern**: All model backends inherit from `BaseServing` in `src/serving/base_serving.py`

**Critical**: `config.yaml` defines:

- `metric_file`: Path to metric implementation (relative to repo root)
- `metric_class`: Class name inheriting from `SeaHelmMetric`
- `metric`: Primary metric for aggregation
- `languages.<lang>.filepath`: Data file path
- `languages.<lang>.prompt_template`: Template with `{fewshot_examples}` placeholder
- `fewshot_num_examples.base` and `fewshot_num_examples.instruct`: Few-shot config
- `aggregation_group`: Groups sub-tasks (e.g., translation) for score averaging

### Model Serving Hierarchy

- `BaseServing` (abstract) → `generate()`, `batch_generate()`, `generate_responses()`
- `VLLMServing`: Offline inference via vLLM engine
- `LiteLLMServing`: API gateway (base for OpenAI, Anthropic, VertexAI)
  - Pass engine args via `--model_args "key=value,key2=value2"` (no spaces!)

**Model Serving Options** (`--model_type`):

- `vllm` - For local HF models (default, most common)
- `openai` - For OpenAI API
- `litellm` - For unified API access to multiple providers
- `anthropic` - For Anthropic API
- `vertexai` - For Google Vertex AI
- `none` - For testing without inference

## Common Workflows

### Running Evaluations

**IMPORTANT**: This project uses `uv` as the package manager and runtime. ALWAYS prefix Python commands with `uv run`:

```bash
# ✅ CORRECT - Always use uv run
uv run python seahelm_evaluation.py --tasks seahelm --output_dir results --model_type vllm --model_name <model>

# ❌ WRONG - Never use python directly
python seahelm_evaluation.py
```

```bash
# Standard vLLM run
uv run python seahelm_evaluation.py --tasks seahelm --output_dir results --model_type vllm \
  --model_name <model_path_or_hf_id> --model_args "tensor_parallel_size=1"

# Base model (skips MT-Bench, Kalahi, IF-Eval)
uv run python seahelm_evaluation.py --tasks seahelm --is_base_model ...

# Reasoning model (DeepSeek-style, temp=0.6)
uv run python seahelm_evaluation.py --tasks seahelm --is_reasoning_model ...
```

**Alternative Scripts**:

- `run_evaluation.sh` - Basic bash wrapper for local execution
- `run_evaluation.pbs` - PBS job scheduler script
- `run_evaluation.slurm` - SLURM job scheduler script

**Key flags**:

- `--tasks`: Task set from `seahelm_tasks/task_config.yaml` (e.g., `seahelm`, `english_evals`)
- `--is_base_model` - Use for base models (skips MT-Bench, Kalahi, IF-Eval). Applies generic base model chat template.
- `--is_reasoning_model` - For DeepSeek-style reasoning models (adds thinking tokens, sets temp=0.6)
- `--is_vision_model` - For vision-capable models (currently unused)
- `--rerun_cached_results` - Force rerun even if cached results exist
- `--limit <n>` - Limit examples per task (TESTING ONLY)

### Adding a New Task

1. Create folder: `seahelm_tasks/<competency>/<task_name>/`
2. Add `config.yaml` following existing patterns (see `instruction_following/ifeval/config.yaml`)
3. Create metric class inheriting `SeaHelmMetric` with `calculate_metrics()` method
4. Place data files in `data/` subdirectory
5. Update `seahelm_tasks/task_config.yaml` to include task in evaluation sets
6. **Version bumping**: Increment `metadata.version` in config for any data/metric changes

### Modifying Existing Tasks

- **ALWAYS** increment the `version` field in `config.yaml`
- Document changes in `readme.md`
- Update aggregation logic in `seahelm_tasks/aggregate_metrics.py` if needed

## Critical Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aisingapore/SEA-HELM](https://github.com/aisingapore/SEA-HELM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
