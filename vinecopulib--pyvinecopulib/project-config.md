---
trigger: always_on
description: handles **discrete** and **zero-inflated** variables. Fitting delegates
---

# AGENTS.md

Normative engineering spec for contributors and coding agents working on
this repository: scope, stability tiers, module boundaries, conventions,
and where to look for what.

For the **mechanics** of the dev workflow (environment setup, Makefile
targets, pre-commit, the CI job graph) see
[CONTRIBUTING.md](CONTRIBUTING.md); for the **user-facing pitch** and
install instructions see [README.md](README.md); for the
release-by-release history see [CHANGELOG.md](CHANGELOG.md). This file does
not duplicate those — it holds the engineering invariants that survive
across pull requests, including the policy half of the branching and
release model.

## Project overview

`pyvinecopulib` is the Python interface to
[vinecopulib](https://github.com/vinecopulib/vinecopulib) — a
header-only C++ library for vine-copula and bivariate-copula
inference, built on Eigen. Two sister C++ libraries ship in the same
wheel:

- [`wdm`](https://github.com/tnagler/wdm) — weighted Kendall's τ /
  Spearman's ρ / Pearson etc.
- [`kde1d`](https://github.com/vinecopulib/kde1d-cpp) — 1-d kernel
  density estimation with boundary correction and discrete support.

All three are vendored as **git submodules under `lib/`** (see
`.gitmodules`). The Python package wraps them through a single
nanobind extension (`pyvinecopulib_ext.cpp`) and adds Python-only
extensions on top:

1. `pyvinecopulib.core`, `pyvinecopulib.families`, `pyvinecopulib.utils`
   — re-exports of the bound C++ surface, organized by topic; `core`
   additionally ships a backend-neutral pair-copula / vine abstraction
   layer (`BicopLike` / `VinecopLike` protocols, `BicopBase` /
   `VinecopBase` canonical bases, `ConditioningContext` policies) that
   custom NumPy / PyTorch backends subclass, and `Vinedist` — a vine
   copula combined with univariate margins, i.e. a full multivariate
   distribution on the data scale.
2. `pyvinecopulib.margins` — the univariate marginal layer `Vinedist`
   composes: the built-in margins, family selection, and an adapter
   registry that presents a SciPy / PyTorch / other-ecosystem
   distribution object as a margin.
3. `pyvinecopulib.sklearn` — scikit-learn-compatible estimators
   (`VineDensity`, `VineRegressor`) on top of the core, with a
   pluggable
   backend layer.
4. `pyvinecopulib.torch` — pure-PyTorch port of the evaluation
   cascade for GPU and autograd workflows.

Three design principles inform the rest of this file:

- **The C++ libraries are upstream.** `lib/vinecopulib`, `lib/wdm`, and
  `lib/kde1d` are git submodules. Behavior changes belong upstream;
  this repo bumps the submodule pin and adjusts the bindings.
- **Generated files are build artifacts.** `src/include/docstr.hpp`
  (libclang-extracted C++ docstrings) and every `.pyi` stub under
  `src/pyvinecopulib/**/__init__.pyi` are gitignored. The build is the
  single source of truth — do not hand-edit, do not commit.
- **Code is quantitatively sensitive.** Pseudo-observation transforms,
  h-functions, Rosenblatt cascades, family parameterizations,
  pickling round-trips, and TLL grids all encode mathematically
  precise behavior. Small "obvious-looking" changes can silently
  break copula identities. Treat numerical paths as
  correctness-critical and prefer round-trip / parity tests over
  structural ones.

### Stability tiers

Different subpackages have different change policies. Honor the tier
when proposing API changes:

| Surface | Tier | Policy |
|---|---|---|
| `pyvinecopulib.core`, `pyvinecopulib.families`, `pyvinecopulib.utils`, top-level `pyvinecopulib` (core class re-exports) | **Stable-ish** | Solid user base. Prefer deprecation aliases over breaks; document migrations in `CHANGELOG.md`. PR #207 is the model: the reorg kept old import paths working via `_deprecations.py` + `DeprecationWarning`. Breaks are allowed (e.g. the pybind11→nanobind migration; the #207 cleanup) but must be intentional, documented, and worth the churn. |
| `pyvinecopulib.margins` | **Active development** | New in the vine-distribution work. The margin contract (`MarginLike` / `MarginBase` in `core`) is the part to treat as load-bearing; the curated parametric family registry, the selection criteria, and the report schema are all expected to move as they meet real data. |
| `pyvinecopulib.sklearn` | **Active development** | API may change in breaking ways between minor releases. The latest break is the `#218` public backend system (estimators now take a single `backend=` instead of loose `controls=`/`structure=`/`seed=` kwargs). |
| `pyvinecopulib.torch` | **Active development** | Same status. Defaults are still being tuned (cf. `990f997` device-aware `batched`, `cache_integrals=True`); the torch↔C++ cascade parity is a hard guarantee, but the `FitControlsTorchVinecop` surface and `TorchVinecop` method signatures may still shift. |
| `pyvinecopulib._python_helpers`, `pyvinecopulib._deprecations` | **Internal** | Underscore-prefixed. Not part of any contract; rename / restructure freely. `_deprecations.py` itself is slated for removal in 2.0. |

The "Solid user base" claim refers to the newest tag (see the
[GitHub project](https://github.com/vinecopulib/pyvinecopulib)).
Unreleased work on `main` is allowed to break sklearn/torch APIs as needed.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vinecopulib/pyvinecopulib](https://github.com/vinecopulib/pyvinecopulib) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
