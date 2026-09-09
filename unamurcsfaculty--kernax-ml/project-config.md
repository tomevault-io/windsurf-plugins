---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Kernax is a JAX-based kernel library for Gaussian Processes, implementing various covariance functions with automatic differentiation and JIT compilation support. The library is built on Equinox and follows a single-class abstract-final pattern, with per-hyperparameter customizable parametrisation.

## Writing Style: No Process Narration

Treat this codebase — including `.md` design/planning docs, docstrings, and code comments —
as final production content, not a record of how we got here.

- Never write comments or doc prose that reference the history of the discussion: no "unlike
  what I wrote before", "contrary to the earlier plan", "corrected from v1", "we used to do X
  but now Y", changelog-style narration inside a design doc, etc. If something changed during
  drafting, just present the current, correct version — don't narrate the correction.
- Don't use code comments or doc text to show your work or prove you understood a correction.
  That belongs in the conversation turn, not in a file a future maintainer will read.
- Explain *why* only when it encodes a non-obvious constraint that will still matter to a
  future reader (a hidden invariant, a real tradeoff, a workaround for an external bug) — not
  "why we don't do it the other way we once considered."
- This applies to design/planning `.md` files too: write them as the current state of the
  plan, not as a diff against previous drafts of themselves.

## Architecture

### Single-Class Pattern

Each kernel is a single class extending `AbstractKernel` (or a subclass like `AbstractStationaryKernel`):

- Stores hyperparameters in **wrapped form** via private fields (e.g., `_length_scale`) with `@property` accessors that unwrap them
- Each hyperparameter has its own **`AbstractParametrisation`** (default: `LogExpParametrisation`) controlling its transformation during optimization
- Implements `pairwise(self, x1, x2)` as an instance method decorated with `@filter_jit`
- Implements `replace(**kwargs)` for immutable parameter updates via `eqx.tree_at()`
- Holds `engine` as a static field (default: `DenseEngine`) controlling matrix construction

### AbstractKernel Base Class

`AbstractKernel` (kernax/AbstractKernel.py) extends `AbstractModule` (which extends `eqx.Module`) and provides:

- **Abstract interface**: Declares `pairwise(self, x1, x2)` and `replace(self, ...)` as abstract methods
- **`__call__`**: Delegates to `self.engine.__call__()`, which handles dimension detection, vectorization, and NaN handling
- **Operator overloading**: `+`, `*`, `-` operators create `SumModule`, `ProductModule`, `NegModule` respectively (defined in `AbstractModule`)

Subclass hierarchy:
- `AbstractStationaryKernel`: adds `distance_function` static field
- `AbstractDotProductKernel`: for dot-product-based kernels

### Kernel Categories

1. **Base Kernels** (implement `pairwise` as instance method):
   - SE (Squared Exponential, aka RBF or Gaussian)
   - Linear, Affine, Polynomial, Sigmoid
   - Matern (1/2, 3/2, 5/2)
   - Periodic, Rational Quadratic
   - Constant, Variance, Feature, WhiteNoise

2. **Operator Modules** (kernax/operators/): Combine two modules
   - `SumModule`: Adds outputs of two modules
   - `ProductModule`: Multiplies outputs of two modules

3. **Wrapper Modules** (kernax/wrappers/): Transform or modify kernel behavior
   - `ExpModule`: Applies exponential
   - `LogModule`: Applies logarithm
   - `NegModule`: Negates output
   - `BatchModule`: Adds batch handling with distinct hyperparameters per batch element
   - `BlockKernel`: Constructs block covariance matrices for grouped data
   - `BlockDiagKernel`: Block-diagonal covariance matrices, specialized version of BlockKernel
   - `ActiveDimsModule`: Selects specific input dimensions before kernel computation
   - `ARDKernel`: Applies Automatic Relevance Determination (different length scale per dimension)

4. **Computation Engines** (kernax/engines.py): Control how covariance matrices are computed
   - `DenseEngine` (default): Computes full covariance matrices
   - `SafeDiagonalEngine`: Returns diagonal matrices (uses conditional check for input equality)
   - `FastDiagonalEngine`: Returns diagonal matrices (assumes x1 == x2, faster but requires constraint)
   - `SafeRegularGridEngine`: Exploits regular grid structure with runtime checks
   - `FastRegularGridEngine`: Exploits regular grid structure without checks (faster but requires constraint)
   - All kernels accept an `engine` parameter for specialized computation patterns

## Development Commands

### Running Python Code
```bash
# Navigate to the kernax directory
cd kernax

# Run Python scripts that import kernax-ml
python3 script.py
```

### Testing Kernels
```bash
# Import and test a kernel in Python REPL
cd kernax
python3
>>> from kernax-ml import SEKernel
>>> import jax.numpy as jnp
>>> kernel = SEKernel(length_scale=1.0)
>>> kernel(jnp.array([1.0]), jnp.array([2.0]))
```

### Running Tests
```bash
# Run all tests
make test

# Run tests with coverage report
make test-cov

# Run tests and generate Allure HTML report
make test-allure

# Run linters (ruff, mypy)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [UNamurCSFaculty/kernax-ml](https://github.com/UNamurCSFaculty/kernax-ml) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
