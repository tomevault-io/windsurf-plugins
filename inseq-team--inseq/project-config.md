---
trigger: always_on
description: **Inseq** is a PyTorch-based interpretability toolkit for analyzing and explaining sequence generation models. It democratizes access to common post-hoc interpretability analyses for generative language models.
---

# CLAUDE.md - Inseq Project Documentation

## Project Overview

**Inseq** is a PyTorch-based interpretability toolkit for analyzing and explaining sequence generation models. It democratizes access to common post-hoc interpretability analyses for generative language models.

- **Version:** 0.7.1
- **Python Support:** 3.10, 3.11, 3.12, 3.13
- **License:** Apache License 2.0
- **Documentation:** https://inseq.org
- **Repository:** https://github.com/inseq-team/inseq

## Quick Start Commands

```bash
# Download and install uv package manager
make uv-download

# Install package (uses uv sync)
make install

# Install with all development dependencies (uses uv sync --all-extras)
make install-dev

# Run all tests
make test

# Run tests without GPU
make test-cpu

# Run fast tests only (skip slow tests)
make fast-test

# Check code style
make check-style

# Auto-format code
make fix-style

# Build documentation
make build-docs

# Serve docs locally
make serve-docs
```

## Dependency Management

This project uses [uv](https://github.com/astral-sh/uv) for fast, reliable Python package management:

- **`uv sync`** - Install core dependencies from `pyproject.toml`
- **`uv sync --all-extras`** - Install all optional dependencies (dev, docs, sklearn, etc.)
- **`uv.lock`** - Lock file ensuring reproducible installations

## Directory Structure

```
inseq/
├── inseq/                    # Main package
│   ├── attr/                 # Attribution methods
│   │   ├── feat/             # Feature attribution implementations
│   │   │   └── ops/          # Custom operations (LIME, DIG, ReAGent, etc.)
│   │   └── step_functions.py # Custom step function registry
│   ├── models/               # Model loading and management
│   │   ├── attribution_model.py    # Base attribution model class
│   │   ├── encoder_decoder.py      # Encoder-decoder support
│   │   ├── decoder_only.py         # Decoder-only support
│   │   └── model_config.yaml       # Model configurations
│   ├── data/                 # Data structures
│   │   ├── attribution.py    # Attribution output classes
│   │   ├── aggregator.py     # Aggregation pipeline
│   │   ├── batch.py          # Batch data structures
│   │   └── viz.py            # Visualization utilities
│   ├── commands/             # CLI implementations
│   │   ├── attribute/        # Single example attribution
│   │   ├── attribute_dataset/# Dataset-wide attribution
│   │   └── attribute_context/# Context dependence detection
│   └── utils/                # Utility modules
├── tests/                    # Test suite (mirrors main package structure)
├── examples/                 # Example notebooks
├── docs/                     # Sphinx documentation
└── pyproject.toml            # Project configuration
```

## Key Modules

### Attribution Methods (`inseq/attr/`)

**Gradient-based:**
- `saliency`, `input_x_gradient`, `integrated_gradients`, `deeplift`, `gradient_shap`
- `discretized_integrated_gradients`, `sequential_integrated_gradients`

**Internals-based:**
- `attention` - Attention weight attribution

**Perturbation-based:**
- `occlusion`, `lime`, `value_zeroing`, `reagent`

### Models (`inseq/models/`)

- `AttributionModel` - Abstract base class
  - `DecoderOnlyAttributionModel` - GPT-2, LLaMA, etc.
  - `EncoderDecoderAttributionModel` - mBART, T5, etc.
- `load_model(model_id, attribution_method)` - Factory function

### Data Structures (`inseq/data/`)

- `FeatureAttributionOutput` - Top-level output container
- `FeatureAttributionSequenceOutput` - Single sequence results
- `Aggregator` / `AggregatorPipeline` - Post-processing chain
- `show_attributions()`, `show_granular_attributions()` - Visualization

### Step Functions (`inseq/attr/step_functions.py`)

Built-in scores: `logits`, `probability`, `entropy`, `crossentropy`, `perplexity`
Contrastive: `contrast_logits`, `contrast_prob`, `pcxmi`
Advanced: `kl_divergence`, `in_context_pvi`, `mc_dropout_prob_avg`

## Architecture Patterns

### Registry Pattern

All extensible components use registries for auto-discovery:

```python
class MyCustomAttribution(FeatureAttribution):
    method_name = "my_method"  # Auto-registered

# List all available
list_feature_attribution_methods()
```

### Input Formatter Protocol

Models implement `InputFormatter` for architecture-specific input handling:

```python
class InputFormatter(Protocol):
    @staticmethod
    def prepare_inputs_for_attribution(...) -> Batch
    @staticmethod
    def format_attribution_args(...) -> tuple
```

### Type System

Extensive type aliases in `utils/typing.py`:

```python
IdsTensor = Int["batch sequence_length"]
LogitsTensor = Float["batch sequence vocab"]
TextInput = str | list[str]
```

## Common API Usage

```python
import inseq

# Load model with attribution method
model = inseq.load_model("gpt2", "saliency")

# Run attribution
out = model.attribute(
    input_texts="The quick brown fox",
    generation_args={"max_new_tokens": 20},
    step_scores=["probability"]
)

# Visualize
out.show()

# Save/load
out.save("result.json", scores_precision="float16")
loaded = inseq.FeatureAttributionOutput.load("result.json")

# Aggregate

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [inseq-team/inseq](https://github.com/inseq-team/inseq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
