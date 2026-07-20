---
trigger: always_on
description: Generates multiple uniformly distributed random parameter points.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Development Commands

```bash
# Install for development
pip install -e .[tests]          # core + test tooling
pip install -e .[dev]            # everything (tests, docs, examples, optional ML backends)

# Run tests
pytest tests/                                           # unit tests
pytest integration_tests/                               # integration tests
pytest tests/.../test_file.py::test_name                # single test (replace path and test name)
pytest --cov emukit --cov-report term-missing tests/    # with coverage
pytest -m 'not (gpy or pybnn or sklearn or notebooks)'  # skip optional-dependency tests
pytest -m gpy                                           # only GPy tests

# Lint and format (enforced in CI) — line length: 120 chars, flake8 exceptions: E731, E127
black .
isort .
flake8 .
```

## Architecture

Emukit is a modular, framework-agnostic library for emulation-based decision-making (Bayesian optimization, experimental design, Bayesian quadrature, sensitivity analysis). The central design is the **OuterLoop**:

```
while stopping_condition not met:
    candidate_point_calculator → next points to evaluate
    user_function(points)       → evaluations
    model_updater               → update model with new data
```

All loop components are swappable, enabling model-agnostic algorithms.

### Key Packages

- **`emukit/core/`** — All shared abstractions:
  - `interfaces/` — Model interfaces (`IModel`, `IDifferentiable`, `IJointlyDifferentiable`, `IPriorHyperparameters`, `IModelWithNoise`)
  - `loop/` — `OuterLoop`, `LoopState`, `CandidatePointCalculator`, `ModelUpdater`, `StoppingCondition`, `UserFunction`, `EventHandler`
  - `acquisition/` — `Acquisition` base class; supports `+`, `*`, `/` operator overloading for composing acquisitions
  - `optimization/` — `AcquisitionOptimizer` (maximizes acquisition over parameter space)
  - `parameter_space.py` — `ParameterSpace` composed of `ContinuousParameter`, `DiscreteParameter`, `CategoricalParameter`, `BanditParameter`
  - `initial_designs/` — Sampling strategies for initialization
  - `encodings.py` — `OneHotEncoding`, `OrdinalEncoding`

- **`emukit/bayesian_optimization/`** — `BayesianOptimizationLoop` (wraps OuterLoop with sensible defaults), acquisitions (EI, EI-MCMC, entropy search, max-value entropy search, local penalization, NegativeLowerConfidenceBound, PoF, PoI)

- **`emukit/experimental_design/`** — `ExperimentalDesignLoop`, design-specific acquisitions

- **`emukit/quadrature/`** — Bayesian quadrature: specialized kernels, loop, and `WarpedBayesianQuadratureModel`

- **`emukit/multi_fidelity/`** — Multi-fidelity GP models built on GPy

- **`emukit/sensitivity/`** — Monte Carlo sensitivity analysis (Sobol indices)

- **`emukit/model_wrappers/`** — Bridges external ML libraries to emukit interfaces: `GPyModelWrapper`, `GPyMultiOutputWrapper`, `SklearnModelWrapper`, `SimpleGaussianProcessModel`

- **`emukit/samplers/`** — MCMC and other samplers

- **`emukit/test_functions/`** — Benchmark functions (Branin, Forrester, etc.)

## Coding Conventions

### Interface Conventions

Interface names are prefixed with `I` (e.g., `IModel`, `IDifferentiable`). Models only need to implement the interfaces required by the algorithms they are used with — there is no single monolithic model class. Type hints are required on all public functions.

### Docstring Style

Use **Sphinx/reStructuredText (reST)** style. Do not use Google style (`Args:`, `Returns:`) or NumPy style (section headers with underlines).

- Parameters: `:param name: description`
- Return value: `:return: description`
- Do not add `:type:` or `:rtype:` tags — types belong in the function signature via type hints only
- Document array shapes inline in the parameter description, e.g. `(n_points x n_dims) array`

```python
def sample_uniform(self, point_count: int) -> np.ndarray:
    """
    Generates multiple uniformly distributed random parameter points.

    :param point_count: number of data points to generate
    :return: Generated points with shape (point_count, num_features)
    """
```

### Optional Dependencies

Optional backends (GPy, pybnn/torch, sklearn) are guarded by `pytest.importorskip()` in tests and declared as optional extras in `pyproject.toml`. Tests for these backends are marked with `@pytest.mark.gpy`, `@pytest.mark.pybnn`, `@pytest.mark.sklearn`, or `@pytest.mark.notebooks`.

### Documentation

API docs are Sphinx-based and live in `doc/`. Each package has a hand-maintained `.rst` file in `doc/api/` that lists its modules via `.. automodule::` directives. Sphinx pulls docstrings from source automatically — but the `.rst` files must be kept in sync with the code structure.

Edit `.rst` files manually — do not use automated tools to regenerate them.

**When to update `doc/api/` `.rst` files:**
- **New file in an existing package**: add a `.. automodule::` block to the corresponding `doc/api/emukit.<package>.rst`:
  ```rst
  .. automodule:: emukit.package.new_module
      :members:
      :undoc-members:
      :show-inheritance:
  ```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EmuKit/emukit](https://github.com/EmuKit/emukit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
