---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

OLMo-core is AI2's training library for the Open Language Model (OLMo) series. It provides modular components for transformer architectures, distributed training, data loading, and evaluation.

## Commands

```bash
# Install (development)
pip install -e '.[all]'

# Run all tests (GPU tests auto-skip without GPU)
pytest -v src/

# Run a specific test file
pytest -v src/test/path/to/test_file.py

# Filter to specific tests by keywords
pytest -v src/test/path/to/test_file.py -k 'keyword'

# Auto-format code
make style

# Check formatting, lint, and types
make checks          # all three at once
make style-check     # isort + black
make lint-check      # ruff
make type-check      # mypy
```

## Code Style

- Line length: 100
- Formatting: `isort` (profile=black) + `black`
- Linting: `ruff` (ignores F403, F405, E501; F401 ignored in `__init__.py`)
- Type checking: `mypy` with `ignore_missing_imports = true`

## Docstrings

- Docstrings should be included on all public classes, methods, and functions.
- We use Sphinx to automatically build API docs by pulling from those docstrings.
- The syntax of the docstrings is a superset of reStructuredText with additional Sphinx-specific syntax for things like:
  - Cross-document links, e.g.:
    ```
    :class:`foo.Foo`  <- links to the class named 'Foo' in the module 'foo'
    :mod:`foo`        <- links to the module named 'foo'
    :func:`foo.bar`   <- links to the function named 'bar' in the module named 'foo'
    ```
  - Documenting parameters (`:param ...:`), return values (`:returns:`), or expected exceptions (`:raises ...:`).

Here's a toy example for a function:

```python
def read_file(path: str) -> str:
    """
    Read a file from disk.

    :param path: The path to the file.

    :returns: The contents of the file.

    :raises FileNotFoundError: If the file doesn't exist.
    """
    pass
```

## Architecture

### Configuration System (`src/olmo_core/config.py`)

Everything is configured via `@dataclass` classes inheriting from `Config`. This is the central design pattern:
- Configs support YAML/JSON serialization, command-line overrides via dot notation (`--train_module.optim.lr=6e-3`), and `merge()` with dotlists.
- The `Registrable` mixin (from `dataclass-extensions`) enables polymorphic config fields — a base config class can resolve to different subclasses at runtime based on a `type` field. Used in optimizers, schedulers, attention backends, and data loaders.
- Nested configs compose modularly: `TrainerConfig` contains `CheckpointerConfig`, `OptimConfig`, etc.

### Training Pipeline (`src/olmo_core/train/`)

- `Trainer` / `TrainerConfig`: Core training loop with checkpointing, evaluation, and an extensible callback system (`callbacks/`).
- `TrainModule`: Wraps the model with forward/backward logic and optimizer. The main concrete implementation is `TransformerTrainModule` / `TransformerTrainModuleConfig`, which handles parallelism setup (DP, TP, PP, CP, EP configs all live here).

### Model Architecture (`src/olmo_core/nn/`)

- `transformer/`: Core transformer with configurable blocks. `TransformerConfig` has factory methods like `olmo2_32B()` for predefined architectures.
- `attention/`: Multi-head attention with backends (flash attention, ring attention, etc.).
- `moe/`: Mixture of Experts with expert parallelism.
- `feed_forward.py`, `layer_norm.py`, `rope.py`, `lm_head.py`: Standard components.

### Data Loading (`src/olmo_core/data/`)

- `NumpyDataset` variants: Memory-mapped numpy datasets for pre-tokenized data (`.npy` files).
- `composable/`: The preferred data loading API, built on a pipeline of `TokenSource` -> `InstanceSource` -> `ComposableDataLoader`. Sources can be sliced, sampled, mixed with ratios, and split for curriculum learning. Use `InstanceSource.visualize()` to inspect the source tree. See the module docstring in `src/olmo_core/data/composable/__init__.py` for detailed examples.
- `mixes/`: Predefined data mixture configs (dolma17, OLMoE-mix-0824, etc.) with paths to tokenized data by source and tokenizer.
- Training data is stored on AI2 infrastructure (Weka filesystem, GCS). For local development, use small validation sets or synthetic data.

### Distributed Training (`src/olmo_core/distributed/`)

- `parallel/`: Implementations of data (FSDP/HSDP/DDP), tensor, pipeline, context (ring attention), and expert parallelism. These can be combined for multi-dimensional parallelism.
- `checkpoint/`: Distributed checkpointing with various filesystem backends.

### Optimization (`src/olmo_core/optim/`)

- Optimizer configs (`AdamWConfig`, `SkipStepAdamWConfig`, `LionConfig`) and LR schedulers (`CosWithWarmup`, etc.).
- `SkipStepOptimizer`: Wrapper for gradient clipping with loss spike detection.

### Examples (src/examples)

Runnable, self-contained examples and reference scripts.

### Training Scripts

Two patterns exist:

**Official scripts** (`src/scripts/official/`): Use `ExperimentConfig` + `main()` from `src/olmo_core/script_utils.py`. Launched with `torchrun` or Beaker. These reproduce published model runs.

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [allenai/OLMo-core](https://github.com/allenai/OLMo-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
