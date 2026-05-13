---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Does

Trains Sparse Autoencoders (SAEs) on neural network activations for mechanistic interpretability. SAEs decompose a frozen LLM's internal activations into sparse, interpretable features using dictionary learning. The LLM is never modified — the SAE is a separate model that learns to explain what the LLM does internally.

## Commands

```bash
# Install
pip install dictionary-learning
# or from source:
pip install poetry && poetry install

# Unit tests (no GPU required, runs in CI)
poetry run pytest tests/unit

# End-to-end tests (requires GPU + ~2 min on RTX 3090)
# These train SAEs on Pythia-70m and compare against expected metrics
poetry run pytest tests/test_end_to_end.py

# Single test
poetry run pytest tests/unit/test_dictionary.py::test_simple_autoencoder
```

## Architecture

### Training Pipeline

The training flow is: **text data → ActivationBuffer → trainSAE() → saved SAE**

1. **Data**: A string iterator (typically from `utils.hf_dataset_to_generator` streaming a HuggingFace dataset)
2. **`ActivationBuffer`** (`buffer.py`): Wraps an `nnsight.LanguageModel`. Runs LM forward passes to collect activations from a target submodule, stores them in a buffer, yields batches. Auto-refills when half-depleted. This is the main training bottleneck — the LM forward pass dominates.
3. **`trainSAE()`** (`training.py`): Main entry point. Takes a buffer + list of trainer configs. Handles the training loop, logging, checkpointing, optional wandb, and activation normalization.
4. **Trainers** (`trainers/`): Each SAE architecture has a paired trainer implementing its specific loss function and update step. All inherit from `SAETrainer` in `trainers/trainer.py`. The trainer's `update(step, activations)` method is the core training step.

### Key Abstractions

- **`Dictionary`** (ABC in `dictionary.py`): All SAE architectures implement `encode(x)`, `decode(f)`, and `forward(x, output_features=False)`. Concrete classes: `AutoEncoder`, `GatedAutoEncoder`, `JumpReluAutoEncoder`, `AutoEncoderTopK`, `BatchTopKSAE`, etc.
- **`SAETrainer`** (`trainers/trainer.py`): Base class. Subclasses implement `update()` and `loss()`. Also defines `ConstrainedAdam` optimizer that constrains decoder weights to unit norm.
- **`ActivationBuffer`** (`buffer.py`): Three variants exist — `ActivationBuffer` (main, text-based), `NNsightActivationBuffer` (token-based), and `HeadActivationBuffer` (per-attention-head, Llama-specific).

### Trainer → Dictionary Mapping

| Trainer | Dictionary Class | Key Hyperparameter |
|---|---|---|
| `StandardTrainer` | `AutoEncoder` | `l1_penalty` |
| `GatedSAETrainer` | `GatedAutoEncoder` | `l1_penalty` |
| `TopKTrainer` | `AutoEncoderTopK` | `k` (active features) |
| `BatchTopKSAETrainer` | `BatchTopKSAE` | `k` |
| `JumpReluTrainer` | `JumpReluAutoEncoder` | `bandwidth` |
| `PAnnealTrainer` | `AutoEncoderNew` | `p_start`, `p_end` |

### Evaluation

`evaluation.py` computes: L2 loss, L1 loss, L0 (active features), variance explained, cosine similarity, L2 ratio, relative reconstruction bias, fraction alive, and **loss recovered** (% of model CE loss preserved when swapping in SAE reconstruction).

### Model Compatibility

Works with any HuggingFace model via `nnsight`. For models that `nnsight` can't load from string (e.g., VLMs like Qwen3.5), load with `AutoModelForCausalLM.from_pretrained()` then wrap: `LanguageModel(torch_model, tokenizer=tokenizer)`.

`utils.py:get_submodule()` has explicit support for: `GPTNeoXForCausalLM`, `Gemma2ForCausalLM`, `Qwen2ForCausalLM`, `Qwen3ForCausalLM`, `LlamaForCausalLM`.

### Qwen-Specific Handling

`ActivationBuffer` has two Qwen-specific parameters:
- `remove_bos=True`: Qwen has no BOS token; this removes the first non-pad token instead
- `max_activation_norm_multiple=10`: Filters high-norm activation sinks that hurt SAE training

### Key Training Options

- `normalize_activations=True` in `trainSAE()`: Normalizes activations to unit mean squared norm. Weights are scaled back before saving so inference doesn't need normalization. Critical for hyperparameter transfer across layers/models.
- `autocast_dtype=t.bfloat16`: Significant speedup with minimal quality impact.
- `lr=None` in TopK trainer configs: Auto-selects LR using `2e-4 / sqrt(dict_size / 2^14)` scaling.
- Typical dict_size: 8x-16x the activation dimension.

## Dependencies

Core: `nnsight>=0.3.0,<0.4.0`, `torch`, `transformers`, `datasets`, `wandb`, `einops`. Managed via Poetry (`pyproject.toml`). Python 3.10+.

## Detailed Documentation (docs/)

The `docs/` directory contains detailed notes organized by topic:

- **Architecture**: [`docs/architecture/overview.md`](docs/architecture/overview.md) — training pipeline, supported architectures, evaluation metrics. [`docs/architecture/model-compatibility.md`](docs/architecture/model-compatibility.md) — nnsight workarounds, VLM loading, Qwen-specific handling.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alif-munim/autosae](https://github.com/alif-munim/autosae) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
