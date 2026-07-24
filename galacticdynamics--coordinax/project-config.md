---
trigger: always_on
description: This is a UV workspace repository containing multiple packages:
---

# Project Overview

This is a UV workspace repository containing multiple packages:

- **coordinax**: Main library for coordinates in JAX with support for JIT compilation, auto-differentiation, vectorization, and GPU/TPU acceleration
- **coordinaxs.api**: Abstract dispatch API that defines the multiple-dispatch interfaces implemented by `coordinax` and other packages. Minimal dependencies (only `plum-dispatch`).
- **coordinaxs.astro**: Astronomy-specific reference frames (ICRS, Galactocentric, etc.) for coordinax
- **coordinaxs.hypothesis**: Hypothesis strategies for property-based testing with `coordinax`

## AUTHORITATIVE SPEC (READ FIRST)

**docs/spec.md is the authoritative source of truth for Coordinax.**

Before making _any_ change You MUST:

1. Read `docs/spec.md` fully.
2. Identify the relevant section(s).
3. Implement code, docstrings, tests, and guides to match the spec exactly.

If existing code conflicts with the spec, the code is wrong. If tests conflict with the spec, the tests are wrong.

If any instruction conflicts with the spec, stop working and identify the problem. New instructions must be given to align with the spec, not to patch around it.

This rule overrides all other instructions.

### Workspace-level specs (CRITICAL)

This repository is a **UV workspace**. Each workspace package has its own authoritative specification file:

- `docs/spec.md` — authoritative for the **coordinax** package
- `packages/coordinaxs.hypothesis/docs/spec.md` — authoritative for the **coordinaxs.hypothesis** package
- (future packages may define their own `docs/spec.md`)

When working inside a workspace package:

- You MUST read and follow that package’s own `docs/spec.md`.
- Before editing any code, read the relevant `docs/spec.md` for the package you are changing.
- If changing behavior, update docstrings, docs, and tests in the same PR.
- If behavior in a package conflicts with its local spec, the code/tests are wrong and must be updated to match the spec.
- Cross-package changes (e.g. coordinax → coordinaxs.hypothesis) MUST keep all relevant specs mutually consistent.

The root `docs/spec.md` defines the **global mathematical framework**; package- local specs define how that framework is instantiated and tested.

## Main Package: coordinax

- **Language**: Python 3.12+
- **Design goals**: JAX-compatible coordinate objects, type-safe transformations, seamless integration with existing JAX code via Quax
- **JAX integration**: Objects are PyTrees via Equinox. Use `quaxed` for pre-quaxified JAX functions. Performant with JIT, vmap, grad.

## Folder Structure

### Root Level (UV Workspace)

- `/src/coordinax/`: Main package. This is a regular package: `src/coordinax/__init__.py` is the user-facing API (`import coordinax as cx`).
- `/packages/`: Workspace packages
  - `coordinaxs.api/`: Abstract dispatch API package
  - `coordinaxs.astro/`: Astronomy-specific frames and transformations
  - `coordinaxs.hypothesis/`: Hypothesis strategies package
  - `coordinaxs.interop.astropy/`: Optional interoperability package for astropy.
- `/tests/`: Main package tests, organized into `unit/`, `integration/`, `benchmark/`
- `README.md`: Main package documentation, tested via Sybil (all Python code blocks are doctests)
- `conftest.py`: Pytest config, Sybil setup, optional dependency handling
- `noxfile.py`: Task automation with dependency groups
- `pyproject.toml`: Root workspace configuration with `[tool.uv.workspace]`

### Main Package Structure (`/src/coordinax/`)

- This is a regular package; the sub-distributions live in the separate `coordinaxs` PEP 420 namespace
- `__init__` : re-export of all the normal functionality in `coordinax` from the other, more specific modules. Main user entry point (`import coordinax as cx`). Most things can be found in here. Other modules are for more specific functionality.
- `angles` : angle-specific functionality, including the `Angle` class and related utilities.
- `distances` : distance-specific functionality, including the `Distance` class and related utilities.

## Coding Style

### Spec Alignment (charts/metrics/frames/embeddings)

**specs files**:

- `docs/spec.md`
- `packages/coordinaxs.hypothesis/docs/spec.md`

Instructions:

- **ALWAYS** read the specs files -- `docs/spec.md` and `packages/coordinaxs.hypothesis/docs/spec.md` -- **before** implementing or changing any chart, metric, frame, embedding code, representation semantics, or conversion rules (`*transform`), and before editing related docs/tests.
- If code behavior and specs file disagree, update code/docstrings/tests to match the specs file.
- Any new public API must be documented in the specs files and preferably referenced from the user guides.

- Treat spec files as executable design documents:
  - read them before coding,
  - cross-reference them in docstrings and comments where appropriate,
  - update them whenever public semantics change.

- Any new transform or representation must include:
  - spec-compliance checklist items (in PR description or doc),
  - concise doctest-like examples where appropriate,
  - property-based tests (prefer `coordinaxs.hypothesis`).

- Never "patch around" a failing spec-driven test. Fix the implementation.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GalacticDynamics/coordinax](https://github.com/GalacticDynamics/coordinax) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
