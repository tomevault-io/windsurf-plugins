---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Sliceline is a Python library for fast slice finding for Machine Learning model debugging. It implements the SliceLine algorithm from the paper "SliceLine: Fast, Linear-Algebra-based Slice Finding for ML Model Debugging" by Svetlana Sagadeeva and Matthias Boehm.

**Core Purpose**: Given an input dataset `X` and a model error vector `errors`, SliceLine identifies the top `k` slices (subspaces defined by predicates) where the ML model performs significantly worse.

## Development Commands

### Environment Setup
```sh
make init                    # Install dependencies via uv
pre-commit install --hook-type pre-push  # Install pre-commit hooks
```

### Testing
```sh
make test                    # Run unit tests with coverage (requires 80% coverage minimum)
uv run pytest                # Run tests without coverage report
uv run pytest tests/test_slicefinder.py::test_experiments  # Run specific test
uv run pytest -k "experiment_1"  # Run tests matching pattern
```

### Code Quality
```sh
uv run ruff check .          # Check code style
uv run ruff format . --check # Check formatting
uv run ruff format .         # Apply formatting
```

### Documentation
```sh
make doc                     # Build Sphinx documentation locally
make notebook                # Start Jupyter notebook server
make execute-notebooks       # Execute all notebooks (run before releases)
```

### Benchmarking

The project includes two types of benchmarks:

**Standalone benchmark scripts** (in `benchmarks/`):
```sh
# Run all benchmarks (cardinality + dataset size scaling)
python benchmarks/benchmarks.py

# Results are saved to:
# - benchmarks/benchmark_results.json (cardinality benchmark)
# - benchmarks/dataset_size_results.json (dataset size benchmark)
```

**pytest-benchmark suite** (in `tests/test_performance.py`):
```sh
# Run performance regression tests with benchmarks
uv run pytest tests/test_performance.py -v --benchmark-only

# Run with full output
uv run pytest tests/test_performance.py -v
```

The standalone benchmarks are for profiling and manual performance analysis.
The pytest-benchmark suite is for regression testing to detect performance regressions.

## Architecture

### Core Algorithm (sliceline/slicefinder.py)

The `Slicefinder` class is a scikit-learn compatible estimator implementing the SliceLine algorithm through sparse linear algebra operations.

**Key Algorithm Steps**:
1. **One-hot encode input**: Convert categorical/numerical features to binary representation
2. **Initialize 1-slices**: Create and score basic slices (single predicates)
3. **Lattice enumeration**: Iteratively combine slices up to `max_l` levels, pruning based on size and error bounds
4. **Top-k maintenance**: Track best slices throughout enumeration

**Critical Parameters**:
- `alpha` (0 < alpha <= 1): Balance between slice size and average error
- `k`: Number of top slices to return
- `max_l`: Maximum predicates per slice (controls combinatorial explosion)
- `min_sup`: Minimum support threshold (absolute or fraction)

**Key Methods**:
- `fit(X, errors)`: Main entry point - searches for slices
- `transform(X)`: Returns binary masks indicating slice membership
- `get_slice(X, slice_index)`: Filters dataset to specific slice
- `_search_slices()`: Core algorithm implementation
- `_score()` / `_score_ub()`: Slice scoring and upper-bound pruning
- `_maintain_top_k()`: Efficiently tracks best slices

**Performance Optimizations (v0.3.0)**:
- Sparse matrix operations (scipy.sparse) throughout
- Direct CSR construction in `_dummify()` (2-3x faster than lil_matrix)
- Sparse-preserving join in `_join_compatible_slices()` (memory efficient)
- Upper-bound pruning to avoid evaluating unpromising candidates
- Missing parent detection to avoid invalid slice combinations
- Deduplication via ID-based hashing
- Deterministic ordering for reproducible results

### Validation Module (sliceline/validation.py)

Custom validation overriding sklearn's `check_array` to **accept string/object dtype inputs** (line 554-555). This is essential because SliceLine works with categorical data that may be represented as strings. The module is derived from sklearn's validation utilities but modified specifically for this use case.

### Numba Optimization Module (sliceline/_numba_ops.py)

Optional JIT-compiled operations for performance improvement. Contains Numba-accelerated versions of:
- `score_slices_numba()`: 5-6x faster slice scoring
- `score_ub_single_numba()` / `score_ub_batch_numba()`: Upper-bound scoring
- `compute_slice_ids_numba()`: ID computation for deduplication

**Coverage exclusion**: This module is excluded from coverage requirements (similar to `validation.py`) because:
1. It's completely optional (only loaded if Numba is installed)
2. Functions are tested indirectly through main slicefinder tests
3. Numba implementations are verified to produce numerically identical results to NumPy fallbacks
4. Direct testing of JIT-compiled functions adds complexity with minimal value

### Testing Structure (tests/)

- `test_slicefinder.py`: Comprehensive unit tests for all private and public methods

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DataDome/sliceline](https://github.com/DataDome/sliceline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
