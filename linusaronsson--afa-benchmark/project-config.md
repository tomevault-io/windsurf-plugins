---
trigger: always_on
description: This guide provides essential information for AI coding agents working with the AFA-Benchmark codebase.
---

# AGENTS.md - Development Guide for AI Coding Agents

This guide provides essential information for AI coding agents working with the AFA-Benchmark codebase.

## Project Overview

**Name:** afa-benchmark
**Description:** A benchmark of active feature acquisition (AFA) methods
**Python Version:** 3.12.10 (exact version required)
**Package Manager:** uv (v0.9.25)
**Main Package:** `afabench/`

## Quick Start Commands

```bash
# Install dependencies
uv sync

# Install pre-commit hooks
pre-commit install

# Run all quality checks (format, lint, type check, test)
just qa

# Run tests
uv run pytest .
```

## Build, Lint & Test Commands

### Formatting & Linting

```bash
# Format code with ruff
uv run ruff format .

# Lint code with auto-fix
uv run ruff check . --fix

# Type check with basedpyright
pre-commit run basedpyright --all-files

# Run all pre-commit hooks
pre-commit run --all-files
```

### Testing

```bash
# Run all tests
uv run pytest .

# Run tests with verbose output
uv run pytest . -v

# Run a single test file
uv run pytest test/path/to/test_file.py

# Run a specific test function
uv run pytest test/path/to/test_file.py::test_function_name

# Run a specific test class
uv run pytest test/path/to/test_file.py::TestClassName

# Run tests with custom options (defined in test/conftest.py)
uv run pytest . --device=cuda           # Use CUDA device
uv run pytest . --cores=4               # Use 4 CPU cores
uv run pytest . -m optional             # Run optional tests
uv run pytest . -m "not optional"       # Skip optional tests (default)
uv run pytest . -m pipeline             # Run pipeline/system tests
uv run pytest . -m "not pipeline"       # Skip pipeline tests (default)
uv run pytest . --no-smoke-test         # Run full tests instead of smoke tests
uv run pytest . --force-rerun           # Force rerun, ignore existing outputs

# Pipeline tests with specific methods and datasets
uv run pytest . -m pipeline --methods jafa ol --datasets cube_without_noise

# Note: Pretrain configs are automatically determined from selected methods
# Example: --methods odin_model_free will only pretrain 'pvae'
#          --methods odin_model_free jafa will pretrain 'pvae' and 'jafa'

# Run coverage analysis
just coverage                           # Generates HTML report in htmlcov/
```

### Dependency Management

```bash
# Add a new dependency
uv add <package>

# Add a development dependency
uv add --dev <package>

# Update lock file
uv lock

# Sync environment with lock file
uv sync
```

## Code Style Guidelines

### General Style

- **Line length:** 79 characters (strictly enforced)
- **Python version:** 3.12.10 exact (use modern Python features)
- **Formatter:** ruff (automatic formatting)
- **Linter:** ruff (ALL rules enabled with specific ignores)
- **Type checker:** basedpyright (recommended mode, relaxed settings)

### Imports

- Import order managed automatically by ruff
- Group imports: standard library, third-party, local
- Implicit namespace packages allowed (no `__init__.py` required everywhere)
- Example:
  ```python
  import os
  from pathlib import Path

  import torch
  from jaxtyping import Float

  from afabench.common.bundle import Bundle
  from afabench.common.registry import Registry
  ```

### Type Annotations

- Type hints encouraged but not strictly required
- Use jaxtyping for tensor shape specifications
- Use Python 3.12+ type alias syntax: `type Features = Float[Tensor, "batch features"]`
- Return type annotations required except for `__init__` methods
- Example:
  ```python
  from jaxtyping import Float
  from torch import Tensor

  type Features = Float[Tensor, "batch features"]
  type Labels = Float[Tensor, "batch"]

  def predict(features: Features) -> Labels:
      ...
  ```

### Naming Conventions

- Follow PEP 8 with some relaxations:
  - Functions/variables: `snake_case`
  - Classes: `PascalCase`
  - Constants: `UPPER_CASE`
  - Type aliases: `PascalCase` (e.g., `Features`, `Labels`)
- N806 (lowercase variables in functions) and N812 (lowercase imports) are ignored where needed

### Error Handling

- Explicit error handling preferred
- Use appropriate exception types
- Logging with f-strings allowed (G004 ignored)
- Example:
  ```python
  import logging

  logger = logging.getLogger(__name__)

  def process_data(data: dict) -> None:
      if "required_field" not in data:
          raise ValueError("Missing required_field in data")
      logger.info(f"Processing data with {len(data)} fields")
  ```

### Documentation

- Docstrings not required for all functions (incrementally adopting)
- Use clear, descriptive function/variable names
- Document complex logic with inline comments
- Key modules should have module-level docstrings

### Testing

- Framework: pytest
- Test files: `test_*.py` in `test/` directory
- Test functions: `test_*`
- Test classes: `Test*`
- Asserts allowed in tests (S101 ignored)
- Use pytest fixtures for setup/teardown
- Mark optional/slow tests with `@pytest.mark.optional`
- Mark pipeline/system tests with `@pytest.mark.pipeline`
- Example:
  ```python
  import pytest

  def test_basic_functionality():
      result = my_function(input_data)
      assert result == expected_output

  @pytest.mark.optional

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Linusaronsson/AFA-Benchmark](https://github.com/Linusaronsson/AFA-Benchmark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
