---
trigger: always_on
description: Enables JAX compatibility for non-native types:
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands for Development

### Code Quality and Linting

ruff is the **only** linter and formatter. Its configuration lives in `[tool.ruff]`
and `[tool.ruff.lint]` in `pyproject.toml`, and it is what `make lint` and the CI
workflow run — there is no second linter and no `.flake8`.

```bash
# Format code with ruff (the formatter)
uv run ruff format src

# Check linting issues with ruff (the linter) -- same as `make lint`
uv run ruff check src

# Fix auto-fixable linting issues
uv run ruff check --fix src

# Run type checking with mypy (`make mypy` resolves the package instead of the path)
uv run mypy -p wax

# Combined quality check
uv run ruff check src && uv run ruff format --check src && uv run mypy -p wax
```

### Testing
```bash
# Run all tests with pytest
uv run pytest src

# Run tests with coverage report
uv run make coverage

# Run single test file
uv run pytest src/wax/stream_test.py

# Run tests with doctests included
uv run pytest --doctest-modules src
```

### Building and Packaging
```bash
# Build package for distribution
uv build

# Install in development mode. `complete` is the union of every feature extra
# (numba, debug, visualization, optional); CI syncs the same set, so a local run
# exercises the same submodules CI does.
uv sync --dev --extra complete
```

### Documentation
```bash
# Build documentation (full build with notebook execution)
make docs

# Build documentation fast (without executing notebooks)
make docs-fast
```

### Makefile targets (what CI runs)

`.github/workflows/tests.yml` calls these targets, so they are the gate:

```bash
# Lint with ruff -- the authoritative linter
make lint

# Type check
make mypy

# Run comprehensive checks: lint, mypy, license, format, coverage, notebooks
make act

# Run tests with coverage
make coverage

# Reformat with ruff and fail if anything changed
make check-format
```

## High-Level Architecture

### Core Design Philosophy
WAX-ML is a **functional programming** library built on JAX with dual backend support (Haiku and Flax) for streaming time-series data processing. It follows a research-oriented design that emphasizes pure functions over object-oriented patterns, with a Flax-based streaming architecture built around a single streaming transform and a library of streaming modules.

### Key Architectural Components

#### 1. Stream Processing (`src/wax/stream.py`)
The heart of WAX-ML's streaming architecture:
- **`Stream` class**: Implements Poincaré-Einstein synchronization for multi-frequency data streams
- **Data tracing mechanism**: Pre-computes indices for efficient JAX-compatible data access
- **Temporal synchronization**: Handles streams with different time resolutions using:
  - Forward-filling for lower frequency data
  - Buffering for higher frequency data
- **Causality preservation**: Ensures no future information leaks into computations

#### 2. Unroll Transformations (`src/wax/unroll.py`)
Generalizes RNN-style sequential processing:
- **`unroll`**: Applies stateful transformations to sequential data
- **`static_scan`**: JAX-optimized scanning for fixed-length sequences
- **Haiku integration**: Works with `transform_with_state` for pure functional state management

#### 3. Data Container Accessors (`src/wax/accessors.py`)
Bridges high-level data APIs with JAX functions:
- **Pandas integration**: `.wax` accessor for DataFrame/Series
- **Xarray integration**: `.wax` accessor for Dataset/DataArray
- **Streaming interface**: `data.wax.stream().apply(function)` pattern
- **Format preservation**: Maintains original data container types in outputs

#### 4. Haiku Modules (`src/wax/modules/`)
Functional building blocks for time-series processing:
- **`EWMA`**: Exponential Moving Average with multiple parameterizations
- **`Buffer`**: Fixed-size buffering for streaming data
- **`UpdateOnEvent`**: Conditional computation updates based on events
- **`OnlineSupervisedLearner`**: Online ML with function optimization
- **Statistical modules**: Rolling statistics, lag operators, differencing

#### 5. Encoding Schemes (`src/wax/encode.py`)
Enables JAX compatibility for non-native types:
- **datetime64 encoding**: Converts to pairs of int32 for JAX compatibility
- **String encoding**: Uses sklearn LabelEncoder for categorical data
- **Reversible transformations**: All encodings support decode operations

#### 6. Flax Streaming Architecture (`src/wax/flax/`)
Streaming computation built on Flax. There is **one** transform, plus a module library:
- **The transform** (`src/wax/flax/core/streaming_transforms.py`):
  `@streaming_transform_with_state` turns a function containing Flax modules into a
  `StreamingTransform` exposing `.init(rng, *args)`, `.apply(params, state, rng, *args)`
  and `.scan(params, state, rng, inputs)`. Everything else in the Flax streaming layer
  is either a module used inside such a function, or a composition built on top of it.
- **Streaming modules** (`src/wax/flax/modules/`): Flax-based equivalents of Haiku modules (EWMA, Buffer, ARMA, etc.)
- **Transform layer** (`src/wax/flax/core/transform.py`): Flax-compatible transform utilities
- **In-tree but not public surface for 0.7.0**: `update_on_event`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eserie/wax-ml](https://github.com/eserie/wax-ml) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
