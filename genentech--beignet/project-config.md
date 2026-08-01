---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Beignet is a standard library for biological research built on PyTorch. It provides specialized operators for computational biology, chemistry, and physics calculations including geometric transformations, orthogonal polynomials, special functions, and molecular analysis tools.

## Development Commands

### Essential Commands
- **Sync dependencies**: `uv sync` (preferred package manager)
- **Run tests**: `uv run python -m pytest`
- **Lint code**: `uv run ruff check` (with auto-fix: `uv run ruff check --fix`)
- **Format code**: `uv run ruff format`
- **Build package**: `uv run python -m build .`
- **Run benchmarks**: `uv run asv run` (see Benchmarking section for details)

### Development Setup
- **Sync all dependencies**: `uv sync` (installs all optional groups automatically)
- **Alternative install for development**: `python -m pip install --editable '.[all]'`
- **Install test dependencies**: `python -m pip install --editable '.[test]'`
- **Install docs dependencies**: `python -m pip install --editable '.[docs]'`

### Benchmarking
- **Run all benchmarks**: `uv run asv run`
- **Run specific benchmarks**: `uv run asv run -b bench_foo`
- **Compare commits**: `uv run asv continuous HEAD~1 HEAD`
- **Generate HTML reports**: `uv run asv publish`
- **View results**: `uv run asv show`

### Documentation
- **Serve docs locally**: `mkdocs serve`
- **Deploy docs**: `mkdocs gh-deploy --force`

## Architecture

### Core Structure
- **src/beignet/**: Main package with ~200+ mathematical and scientific functions
- **beignet.datasets/**: PyTorch datasets for biological and chemical data
- **beignet.features/**: Feature extraction for geometric transformations
- **beignet.structure/**: Protein structure analysis and manipulation
- **beignet.special/**: Special mathematical functions (error functions, integrals)
- **beignet.constants/**: Biological constants and lookup tables

### Key Functional Areas
1. **Geometric Transformations**: Euler angles, quaternions, rotation matrices, transforms
2. **Orthogonal Polynomials**: Chebyshev, Hermite, Laguerre, Legendre polynomials with full mathematical operations
3. **Molecular Analysis**: Protein structure manipulation, trajectory analysis, contact matrices
4. **Scientific Computing**: Numerical integration, root finding, special functions

### Code Organization
- Each function is implemented in a separate file prefixed with underscore (e.g., `_apply_quaternion.py`)
- Functions are imported and exposed through `__init__.py` files
- Test files mirror the source structure in `tests/beignet/`
- Extensive use of PyTorch tensors and operations throughout

### Testing
- Uses pytest with hypothesis for property-based testing
- Test fixtures handle different dtype scenarios (float32/float64)
- Tests located in `tests/beignet/` mirroring source structure
- Run with `python -m pytest` from project root

### Configuration
- **pyproject.toml**: Contains all project configuration including dependencies, build system, and Ruff linting rules
- **Ruff linting**: Configured to use FLAKE8-BUGBEAR, PYCODESTYLE, PYFLAKES, ISORT rules
- **Pre-commit hooks**: Automated formatting and linting on commits
- **GitHub Actions**: Automated testing on multiple Python versions (3.10-3.12) and platforms

### Dependencies
- **Core**: PyTorch, NumPy 2+, biotite, einops, fastpdb, optree, pooch, tqdm
- **Optional datasets**: biopython, lmdb, pandas for extended dataset support
- **Testing**: pytest, hypothesis, pytest-mock, scipy, psutil (for benchmarking)
- **Docs**: mkdocs-material, mkdocstrings

### Benchmarking System
- **ASV (Airspeed Velocity) benchmark suite** for all operators in `benchmarks/`
- **Comprehensive coverage**: Individual benchmarks for all 216+ operators, datasets, and features
- **Categories**: geometric transformations, polynomials, numerical analysis, molecular operations, special functions  
- **Performance metrics**: execution time (time_*) and memory usage (peak_memory_*)
- **torch.compile optimization**: All operators compiled with fullgraph=True for optimal performance
- **Reproducible results**: Seed management via BEIGNET_BENCHMARK_SEED environment variable (default: 42)
- **Parameterized benchmarks**: Multiple batch sizes and dtypes tested
- **ASV commands**:
  - `uv run asv run`: Run all benchmarks
  - `uv run asv continuous`: Compare performance between commits
  - `uv run asv publish`: Generate HTML reports
  - `uv run asv show`: Display benchmark results

## Adding New Operators

When adding new operators to Beignet, follow these guidelines to ensure consistency and quality:

### 1. Module Structure
- **Location**: Place operators at the root level of the package (e.g., `beignet.foo`, not `beignet.submodule.foo`)
- **File naming**: Create a dedicated module file with underscore prefix (e.g., `_foo.py`)
- **Export**: Add the operator to `src/beignet/__init__.py` to make it publicly available:
  ```python
  from ._foo import foo
  __all__ = [..., "foo"]
  ```

### 2. Implementation Requirements

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Genentech/beignet](https://github.com/Genentech/beignet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
