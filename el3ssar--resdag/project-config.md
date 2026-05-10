---
trigger: always_on
description: `resdag` is a **PyTorch-native reservoir computing library** (v0.3.0) for building Echo State Networks (ESNs) and Next Generation Reservoir Computers (NG-RC). It provides GPU-accelerated, modular components for reservoir computing research: stateful reservoir layers, graph-based topology initialization, algebraic readout training, model composition via `pytorch_symbolic`, and Optuna-based hyperparameter optimization.
---

# CLAUDE.md — AI Assistant Guide for ResDAG

## Project Overview

`resdag` is a **PyTorch-native reservoir computing library** (v0.3.0) for building Echo State Networks (ESNs) and Next Generation Reservoir Computers (NG-RC). It provides GPU-accelerated, modular components for reservoir computing research: stateful reservoir layers, graph-based topology initialization, algebraic readout training, model composition via `pytorch_symbolic`, and Optuna-based hyperparameter optimization.

- **Package name**: `resdag`
- **Author**: Daniel Estevez-Moya
- **License**: MIT
- **Homepage**: https://github.com/El3ssar/resdag
- **Python**: >=3.11,<3.15 (classifiers: 3.11–3.14)

---

## Repository Layout

```
resdag/
├── src/resdag/              # Main package (src layout)
│   ├── __init__.py          # Public API + lazy HPO imports; version string
│   ├── composition/
│   │   └── symbolic.py      # ESNModel (extends pytorch_symbolic.SymbolicModel)
│   ├── layers/
│   │   ├── cells/
│   │   │   ├── base_cell.py  # ReservoirCell (abstract single-step interface)
│   │   │   ├── esn_cell.py   # ESNCell — concrete leaky-ESN single-step update
│   │   │   └── ngrc_cell.py  # NGCell — NG-RC feature construction (no weights)
│   │   ├── reservoirs/
│   │   │   ├── base_reservoir.py  # BaseReservoirLayer — sequence loop + state mgmt
│   │   │   ├── esn.py             # ESNLayer — public-facing stateful RNN
│   │   │   └── ngrc.py            # NGReservoir — NG-RC sequence wrapper
│   │   ├── readouts/
│   │   │   ├── base.py       # ReadoutLayer (abstract)
│   │   │   └── cg_readout.py # CGReadoutLayer — CG ridge regression
│   │   └── custom/           # Concatenate, SelectiveExponentiation, Power, etc.
│   ├── init/
│   │   ├── topology/        # Graph topology registry + base classes
│   │   ├── input_feedback/  # Input/feedback weight initializer registry
│   │   ├── graphs/          # NetworkX graph generation functions (17 types)
│   │   └── utils/           # resolve_topology(), resolve_initializer()
│   ├── training/
│   │   └── trainer.py       # ESNTrainer — algebraic readout fitting
│   ├── models/              # Premade architectures
│   │   ├── classic_esn.py
│   │   ├── ott_esn.py       # Ott state-augmented ESN (recommended for chaos)
│   │   ├── power_augmented.py  # Generalized power-augmented ESN
│   │   ├── headless_esn.py
│   │   └── linear_esn.py
│   ├── hpo/                 # Optuna HPO integration (optional dep)
│   │   ├── run.py           # run_hpo()
│   │   ├── losses.py        # efh, horizon, lyap, standard, discounted
│   │   ├── objective.py     # build_objective()
│   │   ├── runners.py       # run_single, run_multiprocess
│   │   ├── storage.py       # Storage backend resolution
│   │   └── utils.py         # get_study_summary, make_study_name, get_best_params
│   └── utils/
│       ├── data/            # load_file(), prepare_esn_data()
│       ├── states/          # esp_index()
│       └── general.py
├── tests/                   # Pytest test suite (mirrors src structure)
├── examples/                # Numbered example scripts (00–10)
├── pyproject.toml           # Build, dependencies, tool configs
├── uv.lock                  # Locked dependency tree
└── .github/workflows/
    └── release.yml          # Auto-release on version tags
```

---

## Development Setup

```bash
# Preferred: uv (faster)
uv sync --dev

# Alternative: pip
pip install -e ".[dev]"

# HPO extras (optional)
pip install -e ".[dev,hpo]"
# or: uv sync --extra hpo
```

**Runtime dependencies**: `torch>=2.10.0`, `numpy>=2.0.0`, `networkx>=3.0`, `pytorch-symbolic>=1.1.1`, `graphviz>=0.21`, `scipy>=1.17.0`

**Dev dependencies**: `basedpyright`, `pytest`, `pytest-cov`, `black`, `ruff`, `mypy`, `optuna`

---

## Running Tests

```bash
# All tests (with coverage)
pytest

# Specific module
pytest tests/test_layers/test_reservoir.py

# Specific test function
pytest tests/test_layers/test_reservoir.py::test_forward_shape

# Without coverage (faster)
pytest --no-cov

# HTML coverage report
pytest --cov=resdag --cov-report=html
```

Test configuration is in `pyproject.toml` under `[tool.pytest.ini_options]`. Tests live in `tests/` and mirror the `src/resdag/` structure. Test files are named `test_*.py`, classes `Test*`, functions `test_*`.

---

## Code Quality

```bash
# Format (line length = 100)
black src/ tests/

# Lint (E, F, I, N, W rules; E501 ignored — handled by black)
ruff check src/ tests/

# Auto-fix lint issues
ruff check --fix src/ tests/

# Type checking
mypy src/
```

**Key formatting rules**:
- Line length: **100 characters** (black + ruff)
- Target: Python 3.11+ syntax (black target `py311`–`py314`, ruff target `py311`)
- `__init__.py` files: unused imports (`F401`) are allowed — they expose the public API

---

## Core Architecture Concepts

### Tensor Conventions

- **3D tensors**: `(batch, timesteps, features)` throughout

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [El3ssar/ResDAG](https://github.com/El3ssar/ResDAG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
