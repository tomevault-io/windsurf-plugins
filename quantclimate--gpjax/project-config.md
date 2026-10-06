---
trigger: always_on
description: When responding locally or writing documents, always use ASD-STE100 Simplified Technical English unless otherwise instructed.
---

# Repository Guidelines

## Communication

When responding locally or writing documents, always use ASD-STE100 Simplified Technical English unless otherwise instructed.

## What is GPJax?

GPJax is a Gaussian process library built on JAX. The API mirrors GP mathematics: you compose a `Prior` (kernel + mean function), multiply it by a `Likelihood` to get a `JointModel`, condition that model on data to get a `Posterior`, then optimise an objective. All modules are Equinox `eqx.Module` subclasses, so they are JAX pytrees and work with `jit`, `vmap`, and `grad`.

## Project Structure & Module Organization
Core GP primitives live in `gpjax/`, with modules such as `kernels/`, `mean_functions.py`, `likelihoods.py`, `variational_families.py`, `conditioning.py`, and `linalg/` mirroring the mathematical decomposition of a GP. Two subpackages hold larger model families: `gpjax/state_space/` (state-space GPs and SDE solvers) and `gpjax/models/` (OILMM). Tests that shadow these modules live in `tests/`—keep filenames aligned (for example, `tests/test_likelihoods.py`) so stack traces map to the right file. Example notebooks are `py:percent` scripts in `docs/examples/` and are rendered into the Sphinx site from `docs/` (output goes to `docs/_build/html`). Benchmarks live in `benchmarks/` (ASV). The root `static/` directory holds the JOSS paper and `CONTRIBUTING.md`; documentation assets live in `docs/static/`. Repo automation resides in the root config files.

## Build, Test, and Development Commands
Create the dev environment with `uv venv && uv sync`. All commands must be prefixed with `uv run`:

```bash
uv run poe test           # pytest with xdist (8 workers), beartype enforcement, skips `slow`
uv run poe test-slow      # the same, including tests marked `slow`
uv run poe lint           # ruff format check + ruff check --fix
uv run poe format         # ruff import sorting + formatting (mutates files)
uv run poe docstrings     # xdoctest on gpjax/
uv run poe all-tests      # lint + docstrings + bench-check + test (CI gate)
uv run poe coverage       # pytest with an XML coverage report for CI
uv run poe docs           # sphinx-build: executes + caches the example notebooks
uv run poe docs-ci        # smoke render, warnings-as-errors (the CI docs gate)
uv run poe docs-serve     # build, then serve docs/_build/html at localhost:8000
uv run poe docs-live      # live-reloading docs server for local authoring
uv run poe docs-linkcheck # sphinx linkcheck, no notebook execution
uv run poe docs-clean     # remove built docs (keeps the notebook exec cache)
uv run poe bench          # run the ASV benchmarks against HEAD
```

Run a single test file or test:
```bash
uv run pytest tests/test_kernels/test_stationary.py -v
uv run pytest tests/test_gps.py::test_conjugate_posterior -v
```

## Architecture

### Core pipeline

```
Prior(kernel, mean_function)  *  Likelihood  -->  JointModel
         |                                            |
    prior(Xtest)                          model.condition(train_data) --> Posterior
         |                                            |
   GaussianDistribution                     posterior(Xtest) --> GaussianDistribution
```

`Prior.__mul__(likelihood)` dispatches through `construct_model()` and returns the correct joint model type:
- `Gaussian` likelihood -> `ConjugateModel` (closed-form conditioning, `ExactPosterior`)
- `Bernoulli`/`Poisson` -> `NonConjugateModel` (latent function values optimised, `LatentPosterior`)
- Heteroscedastic likelihoods carry a second (noise) prior, which the two-operand product cannot express. Build them directly: `HeteroscedasticModel(prior, likelihood, noise_prior=...)`.

`JointModel.predict(test_inputs, train_data)` is a shortcut for `model.condition(train_data)(test_inputs)`.

### Parameter system (`gpjax/parameters.py`)

Parameters are `paramax.AbstractUnwrappable` subclasses. Each class stores its value in an unconstrained internal field and implements `unwrap()` to apply the constraining bijection:

| Class | Bijection | Internal storage |
|---|---|---|
| `Real` | Identity | `value` (unchanged) |
| `PositiveReal` | Softplus | `_unconstrained` via `inv_softplus` |
| `NonNegativeReal` | Softplus | `_unconstrained` via `inv_softplus` |
| `SigmoidBounded` | Sigmoid scaled to `[low, high]` | `_unconstrained` via `logit` |
| `LowerTriangular` | Fill-triangular | `_flat` vector |

**Models are always held in wrapped form.** Wrapping happens once, at construction; nothing unwraps a model tree, including `fit`, which returns a wrapped model. The rule is:

> Call `val(param)` (from `gpjax.parameters`) wherever a parameter meets arithmetic. Pass models around untouched everywhere else.

`val` is idempotent and recursive: it returns the constrained value of a parameter (resolving nested wrappers such as `paramax.non_trainable`) and returns plain arrays or floats unchanged, so it never needs an `isinstance` guard. When writing a new kernel, mean function, likelihood, or objective, every parameter read needs a `val()`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [QuantClimate/GPJax](https://github.com/QuantClimate/GPJax) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
