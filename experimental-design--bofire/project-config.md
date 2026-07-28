---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

BoFire (**B**ayesian **O**ptimization **F**ramework **I**ntended for **R**eal **E**xperiments) is a Python framework for experimental design, combining Design of Experiments (DoE) and Bayesian Optimization (BO). It supports mixed continuous/discrete/categorical parameter spaces, single and multi-objective optimization, and chemical encodings for molecular optimization.

## Build & Test Commands

```bash
# Install for development (full)
pip install -e ".[all]"

# Install core optimization only
pip install -e ".[optimization]"

# Run all tests
pytest tests/

# Run data model tests only (faster, no torch required)
pytest tests/bofire/data_models

# Run with coverage
pytest -ra --cov=bofire --cov-report term-missing tests

# Run a single test file
pytest tests/bofire/strategies/test_sobo.py

# Run a single test
pytest tests/bofire/strategies/test_sobo.py::test_function_name
```

## Linting & Type Checking

```bash
# Install pre-commit hooks (recommended)
pip install pre-commit
pre-commit install

# Run all linting/formatting
pre-commit run --all-files

# Or run ruff directly
ruff check .
ruff format .

# Type checking
ty check bofire
```

## Architecture

### Data Models vs Functional Separation

BoFire separates serializable data models (Pydantic) from functional implementations. This enables REST API integration.

- `bofire/data_models/` - Pydantic models for serialization
- `bofire/strategies/`, `bofire/surrogates/`, etc. - Functional implementations

### Key Modules

- **data_models/** - Pydantic schemas for all BoFire objects
  - `features/` - Input/output feature definitions (ContinuousInput, CategoricalOutput, etc.)
  - `domain/` - Domain composed of Inputs + Outputs + Constraints
  - `objectives/` - MinimizeObjective, MaximizeObjective, CloseToTargetObjective
  - `constraints/` - Linear, nonlinear, and black-box constraints
  - `surrogates/` - Surrogate model specifications
  - `strategies/` - Strategy configurations
  - `acquisition_functions/` - BoTorch acquisition functions (qLogEI, etc.)

- **strategies/** - Optimization strategy implementations
  - Uses ask/tell interface: `strategy.ask(n)` returns candidates, `strategy.tell(experiments)` updates model
  - `botorch/` - BoTorch-based strategies (SoboStrategy, MoboStrategy)
  - `doe/` - Design of Experiments strategies
  - `samplers/` - Sampling methods for constrained spaces

- **surrogates/** - Surrogate model implementations
  - `single_task_gp.py`, `multi_task_gp.py` - Gaussian Processes
  - `random_forest.py`, `mlp.py` - Alternative surrogates

- **kernels/** - GP kernel implementations including molecular kernels

### API Pattern

Each module typically has an `api.py` file that re-exports public interfaces:
```python
from bofire.data_models.features.api import ContinuousInput, CategoricalOutput
from bofire.strategies.api import SoboStrategy
from bofire.data_models.acquisition_functions.api import qLogEI
```

## Data Model Testing

Data models use a spec-based parametrized testing system. The infrastructure lives in `tests/bofire/data_models/specs/`.

### Spec System

The core classes in `tests/bofire/data_models/specs/specs.py`:

- **`Spec(cls, spec_callable)`** — wraps a data model class and a lambda that returns a valid spec dict
- **`InvalidSpec(cls, spec_callable, error, message)`** — wraps an invalid spec with expected error
- **`Specs(invalidators)`** — collection that holds valid/invalid specs; use `add_valid()` and `add_invalid()`

### Serialization Roundtrip Contract

The key invariant enforced by `tests/bofire/data_models/serialization/test_serialization.py`:

```python
spec = some_spec.typed_spec()       # spec dict + {"type": ClassName}
obj = SomeClass(**spec)             # instantiate from spec
assert obj.model_dump() == spec     # EXACT match required
```

**This means:** every field that appears in `model_dump()` output must be present in the spec dict. When adding a new field with a default value to a base class (e.g., `context: Optional[str] = None` on `Feature`), you must add that field with its default to **every `add_valid()` spec** for all subclasses.

### Spec File Patterns

**Leaf specs** (single objects) — spec dicts contain plain values:
```python
# tests/bofire/data_models/specs/features.py
specs.add_valid(
    features.ContinuousInput,
    lambda: {
        "key": str(uuid.uuid4()),
        "bounds": [3, 5.3],
        "unit": None,
        "context": None,   # all fields with defaults must be explicit
    },
)
```

**Container specs** (nested objects) — use `.model_dump()` for children:
```python
# tests/bofire/data_models/specs/domain.py
specs.add_valid(
    Domain,
    lambda: {
        "inputs": Inputs(features=[...]).model_dump(),    # nested via model_dump()
        "outputs": Outputs(features=[...]).model_dump(),
        "constraints": Constraints().model_dump(),
        "context": None,
    },
)
```

Container specs (inputs.py, outputs.py, constraints_container.py, engineered_features.py) don't need manual updates when a field is added to a leaf class, because `.model_dump()` on the nested objects already includes all fields.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [experimental-design/bofire](https://github.com/experimental-design/bofire) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
