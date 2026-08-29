---
trigger: always_on
description: **difflow** is a JAX-based differentiable flowsheet framework for chemical process simulation. It enables automatic differentiation through chemical engineering unit operations for gradient-based optimization, sensitivity analysis, and technoeconomic modeling.
---

# CLAUDE.md - Project Guide for Claude Code

## Project Overview

**difflow** is a JAX-based differentiable flowsheet framework for chemical process simulation. It enables automatic differentiation through chemical engineering unit operations for gradient-based optimization, sensitivity analysis, and technoeconomic modeling.

## Quick Start Commands

```bash
# Install in development mode
pip install -e ".[dev,examples]"

# Run tests
pytest tests/ -v

# Run specific test file
pytest tests/test_cstr.py -v

# Run tests with coverage
pytest tests/ --cov=src/difflow

# Build documentation (Jupyter Book)
make book

# Execute all example notebooks
make notebooks
```

## Repository Structure

```
difflow/
├── src/
│   ├── difflow/           # Core package
│   │   ├── streams.py     # Stream representation
│   │   ├── thermo.py      # Thermodynamics (ideal)
│   │   ├── eos.py         # Equations of State (PR, SRK)
│   │   ├── database.py    # Species property database
│   │   ├── flowsheet.py   # Flowsheet with recycle solving
│   │   ├── uncertainty.py # Sensitivity & UQ
│   │   ├── planning/      # Delta-base planning (LP/MILP + trust region)
│   │   ├── catalog.py     # Machine-readable schema of every unit operation
│   │   ├── serialize.py   # Flowsheet <-> JSON round trip
│   │   ├── codegen.py     # Flowsheet -> runnable Python source
│   │   ├── kinetics.py    # Declarative mass-action rate laws (data, not callables)
│   │   ├── publish.py     # Flowsheet -> self-contained interactive HTML (no install)
│   │   ├── gui.py         # Local browser editor (python -m difflow.gui)
│   │   ├── params_mixin.py # ParamsMixin base class for Params dataclasses
│   │   ├── reconciliation/ # Data reconciliation, gross error detection,
│   │   │                   # observability, monitoring, multi-set pooling
│   │   ├── units/         # Steady-state unit operations
│   │   ├── dynamic/       # Dynamic modeling (DAE)
│   │   ├── economics/     # Technoeconomic analysis
│   │   └── visualization/ # Flowsheet visualization
│   ├── difflow_bio/       # Bio manufacturing plugin (bioreactors, filtration, chromatography)
│   ├── difflow_ree/       # Rare earth element solvent extraction plugin
│   ├── difflow_cc/        # Carbon capture plugin (amine, membrane, adsorption)
│   └── difflow_gas/       # Gas transmission network plugin (pipes, compressors, computed decomposition)
├── tests/                 # pytest test files (includes tests/bio/, tests/ree/, tests/cc/, tests/gas/)
├── examples/              # Jupyter notebook examples
├── jax-tutorials/         # JAX/autodiff tutorials
└── docs/                  # Documentation (Markdown)
```

## Key Concepts

### 1. Streams
Streams are JAX-compatible data structures representing material flows:
```python
from difflow import Stream, create_experiment_stream

# Create a stream
stream = create_experiment_stream(
    conditions={'T': 350.0, 'P': 101325.0},
    species=['A', 'B'],
    molar_flows=[1.0, 0.5]
)
```

### 2. Unit Operations and Params Classes
All units are differentiable and use Params dataclasses that inherit from `ParamsMixin`:
```python
from difflow import CSTR, CSTRParams
import jax.numpy as jnp

# Define rate function: A -> B, r = k*C_A
def rate_fn(concentrations, T, params):
    k = params['k'] * jnp.exp(-params['Ea'] / (8.314 * T))
    return k * concentrations['A']

# Create params with dict-like access via ParamsMixin
params = CSTRParams(
    V=1.0,  # Reactor volume (m^3)
    rate_fn=rate_fn,
    stoich={'A': -1, 'B': 1},
)

# Create and run CSTR
cstr = CSTR(params)
outlet = cstr(inlet_stream)

# Params support dict-like access
print(params['V'])        # -> 1.0
print('V' in params)      # -> True
new_params = params.update(V=2.0)  # Functional update (JAX-compatible)
```

### 3. ParamsMixin Pattern
All `Params` dataclasses should inherit from `ParamsMixin` for consistent API:
```python
from dataclasses import dataclass
from difflow.params_mixin import ParamsMixin

@dataclass
class MyUnitParams(ParamsMixin):
    """Parameters for MyUnit.

    Attributes:
        temperature: Operating temperature (K)
        pressure: Operating pressure (Pa)
    """
    temperature: float
    pressure: float

# ParamsMixin provides:
# - params['key'] - dict-style access
# - params.update(key=value) - JAX-compatible functional updates
# - params.keys(), .values(), .items() - dict-like iteration
# - 'key' in params - membership testing
# - Concise __repr__ with JAX array formatting
```

### 4. Automatic Differentiation
Use JAX's `grad`, `jacobian`, `jit` with any difflow function:
```python
import jax
from jax import grad, jit

def conversion(volume):
    params = CSTRParams(V=volume, rate_fn=rate_fn, stoich=stoich)
    cstr = CSTR(params)
    outlet = cstr(inlet)
    return outlet.molar_flows['B'] / inlet.molar_flows['A']

# Gradient of conversion w.r.t. volume
d_conv_d_V = grad(conversion)(1.0)

# JIT compile for speed
fast_conversion = jit(conversion)
```

### 5. Flowsheets with Recycles
```python
from difflow import Flowsheet

fs = Flowsheet()
fs.add_unit('cstr', cstr)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jkitchin/differentiable-flowsheets](https://github.com/jkitchin/differentiable-flowsheets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
