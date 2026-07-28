---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Structure

LBSTER (Lobster) is a "batteries included" language model library for proteins and biological sequences built with PyTorch Lightning and Hydra configuration management. The project uses a modular architecture with clear separation between data handling, model definitions, training, and evaluation.

### Key Directories

- `src/lobster/` - Main package containing all core functionality
- `src/lobster/model/` - Model architectures (MLM, CLM, concept bottleneck models, etc.)
- `src/lobster/data/` - Data modules for various biological datasets
- `src/lobster/datasets/` - Dataset implementations for different data types
- `src/lobster/tokenization/` - Tokenizers for biological sequences (amino acids, nucleotides, SMILES)
- `src/lobster/transforms/` - Data transformation functions
- `src/lobster/evaluation/` - Model evaluation tools and benchmarks (DGEB integration, callbacks)
- `src/lobster/hydra_config/` - Hydra configuration files for all components
- `src/lobster/cmdline/` - Command-line interface implementations
- `tests/` - Unit tests mirroring the src structure
- `examples/` - Example scripts for inference and interventions
- `notebooks/` - Jupyter notebooks for tutorials

## Development Commands

### Essential Commands
- **Sync dependencies**: `uv sync` (preferred package manager)
- **Run tests**: `uv run python -m pytest`
- **Lint code**: `uv run ruff check` (with auto-fix: `uv run ruff check --fix`)
- **Format code**: `uv run ruff format`
- **Build package**: `uv run python -m build .`

### Development Setup
- **Sync all dependencies**: `uv sync --all-extras`
- **Alternative install for development**: `python -m pip install --editable '.[all]'`

## Common Commands

### Installation and Environment
```bash
# Primary installation method with uv
uv sync
uv sync --extra flash  # With flash attention

# Alternative with mamba
mamba env create -f env.yml
pip install -e .
```

### Testing
```bash
python -m pytest -v --cov-report term-missing --cov=./lobster ./tests
```

### Code Quality
```bash
pre-commit install  # Setup pre-commit hooks
pre-commit run --all-files  # Run all checks
ruff check --fix  # Lint and fix issues
ruff format  # Format code
```

### Code Organization
- Each function is implemented in a separate file prefixed with underscore (e.g., `_model.py`)
- Functions are imported and exposed through `__init__.py` files
- Test files mirror the source structure in `tests/lobster/`
- Extensive use of PyTorch tensors and operations throughout

### Testing
- Test fixtures handle different dtype scenarios (float32/float64)
- Tests located in `tests/lobster/` mirroring source structure
- Run with `uv run pytest` from project root

### Configuration
- **pyproject.toml**: Contains all project configuration including dependencies, build system, and Ruff linting rules
- **Ruff linting**: Configured to use FLAKE8-BUGBEAR, PYCODESTYLE, PYFLAKES, ISORT rules
- **Pre-commit hooks**: Automated formatting and linting on commits
- **GitHub Actions**: Automated testing on multiple Python versions (3.10-3.12) and platforms

### Documentation
- **Docstring format**: Follow NumPy-style docstrings used throughout the project

### Code Quality
- **Type hints**: Include comprehensive type annotations for all parameters and return values
- **Input validation**: Validate input shapes and types, raising appropriate errors
- **Error messages**: Provide clear, actionable error messages
- **Code style**: Follow the project's linting rules (run `uv run ruff check` and `uv run ruff format`)

### Training and Inference
```bash
# Train a model
lobster_train data.path_to_fasta="test_data/query.fasta" logger=csv paths.root_dir="."

# Embed sequences
lobster_embed data.path_to_fasta="test_data/query.fasta" checkpoint="path_to_checkpoint.ckpt"

# Other CLI commands
lobster_predict
lobster_intervene
lobster_perplexity
lobster_eval
lobster_dgeb_eval ume-mini-base-12M --modality protein
```

## Model Architecture

The library implements several model types:

1. **LobsterPMLM**: Masked Language Model (BERT-style encoder-only)
2. **LobsterCBMPMLM**: Concept Bottleneck Masked Language Model with 718 biological concepts
3. **LobsterPCLM**: Causal Language Model (Llama-style decoder-only)
4. **LobsterPLMFold**: Structure prediction models (encoder + structure head)
5. **Modern BERT variants**: Including FlexBERT and Hyena architectures

### Key Base Classes
- `LMBaseForMaskedLM` - Base class for masked language models
- `LMBaseContactPredictionHead` - Contact prediction head for structure tasks
- Models extend PyTorch Lightning modules for training management

## Configuration System

The project uses Hydra for configuration management:

- All configs in `src/lobster/hydra_config/`
- Modular configs: `data/`, `model/`, `callbacks/`, `trainer/`, etc.
- Default training config uses `fasta` data and `mlm` model
- Override configs with command-line syntax: `model=clm data=calm`

## Tokenization

Multiple specialized tokenizers for biological sequences:
- `pmlm_tokenizer` - Default protein tokenizer
- `amino_acid_tokenizer` - Amino acid sequences

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prescient-design/lobster](https://github.com/prescient-design/lobster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
