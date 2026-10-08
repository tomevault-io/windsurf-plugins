---
trigger: always_on
description: This file provides guidance to AI coding agents working in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents working in this repository.

## Project Overview

Impulso is a Python library for Bayesian Vector Autoregression (VAR). Early stage, requires Python >=3.11.

## Commands

```bash
# Install dependencies and prek hooks
make install

# Run all code quality checks (lock file, pre-commit, type checking)
make check

# Run tests with coverage
make test

# Run a single test
uv run python -m pytest tests/test_foo.py::test_foo -v

# Run only fast tests (skip MCMC sampling)
uv run python -m pytest -m "not slow" -v

# Type check only
uv run ty check

# Lint/format only
uv run ruff check . && uv run ruff format .

# Build the Sphinx/MyST-NB docs (executes + caches notebooks; full MCMC)
make docs

# Build docs in smoke mode, warnings-as-errors (fast MCMC; the PR CI gate)
make docs-ci

# Serve the built docs locally on :8000
make docs-serve

# Check external links resolve (no notebook execution)
make docs-linkcheck

# Multi-version test (Python 3.11-3.14)
uv run tox
```

## Architecture

- **Source**: `src/impulso/` — library code, built as a wheel via Hatchling
- **Tests**: `tests/` — pytest with `--cov`, 90% coverage target (codecov.yaml)
- **Docs**: `docs/` — Sphinx + MyST-NB (`docs/conf.py`), tutorials as executable **jupytext `py:percent` `.py` notebooks** (read via `nb_custom_formats`), prose/reference pages as MyST `.md`. Docstrings auto-rendered via Sphinx `autosummary` (summary tables + per-object pages generated under `docs/reference/generated/`, gitignored; custom stub templates in `docs/_templates/autosummary/` emit MyST) + `napoleon`. `sphinx-codeautolink` links API names in tutorial code to the reference; `sphinxcontrib-bibtex` powers `{cite}` (bib in `docs/references.bib`); `sphinx-sitemap` emits `sitemap.xml`; `numfig` + labelled `$$…$$ (label)` equations cross-referenced with `` {eq}`label` ``. Link-preview (Open Graph) tags come from `sphinxext-opengraph`; a page opts into a custom `og:image` via a `meta` directive in its first markdown cell. Notebooks execute at build via jupyter-cache; no outputs are committed. Theme: `shibuya`. Tutorial `.py` files are excluded from ruff (they are notebooks, not library code).

### Core Pipeline

The library follows an immutable pipeline where each stage produces the next:

```
VARData → VAR.fit() → FittedVAR → .set_identification_strategy() → IdentifiedVAR
```

- **`VARData`** (`data.py`): Validated, frozen Pydantic model holding endogenous/exogenous arrays + DatetimeIndex. Constructor `from_df()` for DataFrame input.
- **`VAR`** (`spec.py`): Model specification (lags, prior). Calling `.fit(data, sampler)` builds a PyMC model, samples, and returns `FittedVAR`.
- **`FittedVAR`** (`fitted.py`): Reduced-form posterior. Provides `.forecast(steps)` → `ForecastResult` and `.set_identification_strategy(scheme)` → `IdentifiedVAR`.
- **`IdentifiedVAR`** (`identified.py`): Structural VAR. Provides `.impulse_response()` → `IRFResult`, `.fevd()` → `FEVDResult`, `.historical_decomposition()` → `HistoricalDecompositionResult`.

### Extensibility via Protocols

Three `Protocol` classes in `protocols.py` define the extension points:

- **`Prior`**: must implement `build_priors(n_vars, n_lags, *, sigma) → dict`. Concrete: `MinnesotaPrior`.
- **`Sampler`**: must implement `sample(model) → InferenceData`. Concrete: `NUTSSampler`.
- **`IdentificationScheme`**: must implement `identify(idata, var_names) → InferenceData`. Concrete: `Cholesky`, `SignRestriction`.

### Result Objects

All result types (`results.py`) inherit from `VARResultBase` and provide `.median()`, `.hdi()`, `.to_dataframe()`, and `.plot()`. Plot methods delegate to `plotting/` subpackage.

### Base Model Classes (`_base.py`)

Two base classes centralise Pydantic config:

- **`ImpulsoBaseModel`**: `frozen=True, arbitrary_types_allowed=True` — for models holding numpy arrays, InferenceData, etc.
- **`ImpulsoModel`**: `frozen=True` only — for models with standard Python types (e.g., `Cholesky`, `SignRestriction`).

All domain models inherit from one of these. Use `object.__setattr__` only for internal setup (e.g., making arrays read-only in validators).

### Key Patterns

- **Lazy imports**: PyMC, plotting modules, and cross-module types are imported inside methods (or guarded by `TYPE_CHECKING`) to avoid circular imports and reduce import time.
- **Prior registry**: `spec.py` maps string shorthands (e.g., `"minnesota"`) to `Prior` classes via `_PRIOR_REGISTRY`.
- **Runtime type checking**: `impulso.enable_runtime_checks()` wraps public API classes with beartype decorators. Intended for test suites.
- **`select_lag_order(data, max_lags)`**: OLS-based information criteria (AIC/BIC/HQ) for choosing VAR lag order — fast, no MCMC.

## Tooling

- **Package manager**: uv (lock file must stay in sync — `uv lock --locked`)
- **Linter/Formatter**: Ruff — line length 120, target py311, auto-fix enabled
- **Type checker**: ty (configured for `.venv`, Python 3.11). Ignores `unresolved-attribute`, `not-subscriptable`, and `invalid-argument-type` due to ArviZ/PyMC/pandas dynamic attrs.
- **Prek**: Ruff checks, vulture (dead-code check, `[tool.vulture]` + `vulture_whitelist.py`), and standard hooks (trailing whitespace, TOML/YAML/JSON validation)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [QuantClimate/Impulso](https://github.com/QuantClimate/Impulso) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
