---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OneShotGRPO is a modular package for training small language models on math reasoning tasks using GRPO (Generative Reinforcement Policy Optimization). The codebase provides both programmatic APIs and CLI tools for fine-tuning models on GSM8K with structured XML-formatted chain-of-thought reasoning.

**Core Concept**: Train models to output both reasoning steps and answers in a specific XML format:
```xml
<reasoning>
[step-by-step problem solving]
</reasoning>
<answer>
[numeric answer]
</answer>
```

## Development Commands

### Installation
```bash
# Install the package in editable mode
pip install -e .

# Core dependencies
pip install datasets transformers trl wandb

# For optimal training performance
pip install vllm  # MUST be installed BEFORE trl
```

### Training

**CLI Training** (recommended for production):
```bash
python -m oneshot_grpo.train.run_math_grpo \
  --model-name Qwen/Qwen2.5-0.5B-Instruct \
  --use-wandb \
  --wandb-project qwen-math-grpo \
  --wandb-entity your-entity \
  --push-to-hub \
  --hub-model-id your-hf-user/model-name
```

**Programmatic Training**:
```python
from oneshot_grpo.train import create_math_training_setup, run_training

setup = create_math_training_setup(
    model_name="Qwen/Qwen2.5-0.5B-Instruct",
    split="train",
)
run_training(setup.artifacts)
```

### Inference

**Using trained model**:
```python
from oneshot_grpo.inference import build_text_generator, generate_math_solution

generator = build_text_generator(model_id="your-hf-user/model-name")
solution = generate_math_solution(
    "What is 25 * 4?",
    generator=generator,
    max_new_tokens=200,
    temperature=0.9
)
```

## Architecture

### Package Structure

```
src/oneshot_grpo/
├── data/           # Dataset loading and preprocessing
├── rewards/        # Reward functions for RL training
├── train/          # Training orchestration
├── inference/      # Inference utilities
└── eval/           # Evaluation (placeholder)
```

### Key Components

**1. Data Pipeline** ([data/gsm8k.py](src/oneshot_grpo/data/gsm8k.py))
- `load_gsm8k()`: Loads GSM8K dataset from HuggingFace
- `format_for_grpo()`: Transforms raw data into GRPO-compatible format with chat templates
- `extract_hash_answer()`: Parses numeric answers from GSM8K's `#### [answer]` format
- `build_chat_prompt()`: Creates system + user message pairs with XML format instructions

**2. Reward System** ([rewards/math.py](src/oneshot_grpo/rewards/math.py))

Multi-signal reward stack (total 1.2 points possible):
- **Correctness** (1.0): Binary reward for matching ground truth (primary signal)
- **Numeric validation** (0.1): Ensures answer is numeric
- **XML format** (0.1): Partial credit for XML tag presence

Reward functions accept TRL's signature: `(prompts, completions, reference_answer, **kwargs) -> List[float]`

**3. Training Pipeline** ([train/math_pipeline.py](src/oneshot_grpo/train/math_pipeline.py))

- `create_math_training_setup()`: One-stop setup function that:
  1. Loads and formats GSM8K dataset
  2. Configures default reward functions
  3. Builds GRPOTrainer with optimized hyperparameters
  4. Returns `MathTrainingSetup` with all artifacts

- Default hyperparameters (see `DEFAULT_GRPO_ARGS`):
  - Learning rate: 5e-6 (appropriate for LLM fine-tuning)
  - Cosine LR schedule with 10% warmup
  - Single epoch to prevent overfitting
  - 16 generations per prompt for policy exploration
  - Gradient accumulation: 4 steps (effective batch size = 4)

**4. Inference** ([inference/pipeline.py](src/oneshot_grpo/inference/pipeline.py))

- `build_text_generator()`: Creates HF pipeline for trained model
- `generate_math_solution()`: High-level inference with sensible defaults
- `save_prompt_and_response()`: JSONL logging for synthetic data generation

### Training Flow

1. **Dataset Preparation**: GSM8K → chat format with system prompt
2. **Model Generation**: Produce 16 completions per prompt
3. **Reward Computation**: Evaluate each completion across 3 reward functions
4. **Policy Update**: GRPO updates based on weighted reward signal
5. **Checkpointing**: Save every 100 steps to `outputs/{run_name}/checkpoint-{step}`

### Important Implementation Details

**vLLM Integration**:
- vLLM MUST be installed before TRL (runtime dependency conflict)
- Used for efficient batched inference during training
- Default GPU memory allocation: 30% for vLLM, 70% for training
- To enable vLLM in training, set `use_vllm=True` in GRPOConfig

**Answer Extraction Logic** ([rewards/math.py:13-21](src/oneshot_grpo/rewards/math.py)):
- Primary: XML tag extraction `<answer>...</answer>`
- Fallback: GSM8K hash format `#### [number]`
- Supports both exact string matching and numeric comparison with tolerance

**Dataset Fields** (after `format_for_grpo`):
- `prompt`: List of chat messages (system + user)
- `reference_solution`: Full GSM8K solution with reasoning
- `reference_answer`: Extracted numeric answer for reward computation

**Reward Function Signature**:
```python
def reward_func(
    *,
    prompts: Sequence[str],
    completions: Sequence[Any],
    reference_answer: Sequence[str],

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HarleyCoops/OneShotAquaRAT](https://github.com/HarleyCoops/OneShotAquaRAT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
