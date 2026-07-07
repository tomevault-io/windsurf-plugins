---
trigger: always_on
description: Guidance for AI coding agents and human contributors working on VersatIL.
---

# AGENTS.md

Guidance for AI coding agents and human contributors working on VersatIL.

## Working Agreements

### Docstrings

Describe what the code does, what each non-obvious argument means, what it returns, and what it raises. Do not explain behavior by saying what it is not, unless that contrast is necessary to prevent misuse.

### Code Quality

**Write Human Code.** Write code that reads like a human wrote it. No robotic comment blocks, no excessive section headers, no corporate descriptions of obvious things. If three experienced devs would all write it the same way, that's the way.

**Don't Over-Engineer.** Don't build for imaginary scenarios. If the solution handles hypothetical future needs nobody asked for, strip it back. Simple and correct beats elaborate and speculative.

**Demand Elegance (Balanced).** For non-trivial changes: pause and ask "is there a more elegant way?" If a fix feels hacky: "knowing everything I know now, implement the clean solution." Skip this for simple, obvious fixes. Challenge your own work before presenting it.

**Stand Ground.** Do not reflexively validate user claims. If a user premise is technically wrong, incomplete, or unsupported by the code, say so directly and explain the correction briefly. Agreement should be reserved for claims that are actually correct.


## Project Overview

VersatIL: Imitation Learning framework for robotic manipulation. The codebase provides a modular architecture in `src/versatil/`. All new development should target the versatil package.

**Goal**: Develop all new code in the modular design in `src/versatil/`.

## Environment Setup

```bash
# Option A: Create a conda environment (Mamba recommended for faster installation)
mamba env create -f environment.yml
mamba activate versatil
PYTHON_VERSION=3.14
UV_PROJECT_ENVIRONMENT=$CONDA_PREFIX uv sync --python "$PYTHON_VERSION" --extra gpu
# For CPU-only environments:
# UV_PROJECT_ENVIRONMENT=$CONDA_PREFIX uv sync --python "$PYTHON_VERSION" --extra cpu
# For Python 3.13, set PYTHON_VERSION=3.13.

# Option B: Create a uv-managed .venv without conda/mamba
PYTHON_VERSION=3.14
uv python install "$PYTHON_VERSION"
uv venv --python "$PYTHON_VERSION"
source .venv/bin/activate
uv sync --python "$PYTHON_VERSION" --extra gpu
# For CPU-only environments:
# uv sync --python "$PYTHON_VERSION" --extra cpu

# Optional ExecuTorch support through PyPI, available on Python 3.13.
PYTHON_VERSION=3.13
uv sync --python "$PYTHON_VERSION" --extra cpu --extra executorch

# Install pre-commit hooks (required for all contributors)
pre-commit install
```

Requirements: Python 3.13 or 3.14. CUDA is optional; the `gpu` extra installs
the CUDA 13.0 PyTorch wheel set.

`uv sync` installs the `dev` dependency group (pytest, pytest-cov, ruff,
pre-commit) by default. Pass `--no-dev` for a runtime-only install.

## Common Commands

### Running Tests

```bash
# Run unit tests only (default)
pytest

# Run all tests including integration tests
pytest -m ""

# Run specific test file
pytest tests/data/test_dataloader.py

# Run tests by marker
pytest -m "unit"           # Fast unit tests with mocked dependencies
pytest -m "integration"    # Slower tests with real model downloads
pytest -m "requires_gpu"   # GPU-required tests
pytest -m "not slow"       # Skip slow tests
```

### Training

```bash
# Train with an end-to-end config (Hydra)
python -m versatil.endpoints.train --config-name end_to_end_training_runs/bowel_retraction/act

# Override parameters from CLI
python -m versatil.endpoints.train \
    --config-name end_to_end_training_runs/bowel_retraction/act \
    task.dataloader.batch_size=64 training.optimizer.lr=1e-4

# Override a defaults list entry (e.g. swap dataset schema)
# Use slash syntax (group override), NOT dot syntax (value override)
python -m versatil.endpoints.train \
    --config-name end_to_end_training_runs/synthetic/bcat \
    task/dataset_schema=synthetic/conditional_circle

# Resume from checkpoint
python -m versatil.endpoints.train \
    --config-name end_to_end_training_runs/bowel_retraction/act \
    experiment.resume_from=/path/to/checkpoint.ckpt
```

### Post-Training Compression

```bash
# Compress a trained checkpoint with default x86 PT2E config
python -m versatil.endpoints.post_training_compress \
    --config-name end_to_end_ptq/unstructured_prune_x86.yaml \
    checkpoint_path=/path/to/training/checkpoint

# Override pruning amount and calibration steps
python -m versatil.endpoints.post_training_compress \
    --config-name end_to_end_ptq/unstructured_prune_x86 \
    checkpoint_path=/path/to/checkpoint \
    calibration_steps=32 \
    generate_report=true

# Run compressed model inference
python -m versatil.endpoints.deploy \
    checkpoint_path=/path/to/checkpoint/compressed/<timestamp> \
    device=cpu \
    client.model_server_address=10.0.0.1 \
    client.model_server_port=5556
```

### Code Formatting and Linting

```bash
# Format code with Ruff (line length 88)
ruff format src/ tests/

# Check formatting
ruff format --check src/ tests/

# Lint
ruff check src/ tests/

# Lint and auto-fix
ruff check --fix src/ tests/
```

## VersatIL Architecture (`src/versatil/`)

The modular design separates concerns into composable components configured via Hydra.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lorenzo-Mazza/VersatIL](https://github.com/Lorenzo-Mazza/VersatIL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
