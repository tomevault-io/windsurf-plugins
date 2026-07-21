---
trigger: always_on
description: `ridgeplot` is a Python package for beautiful, interactive ridgeline plots built on Plotly.
---

# ridgeplot Development Guide for AI Agents

`ridgeplot` is a Python package for beautiful, interactive ridgeline plots built on Plotly.

## Start Here

- Read this file first, then the most relevant source file(s).
- Prefer local docs and tests over memory; do not guess behavior.
- If requirements are unclear or risky, ask a concrete question before changing code.

## Stack and Style Constraints

- Python >=3.10
- Plotly graph objects (dependency is `plotly>=5.20`)
- Line length 100, formatting with ruff
- Docstrings are NumPy style
- Type annotations are modern and throughout with strict checking via pyright

## Quick Commands

### Environment Setup

```shell
# Initialize full development environment (recommended for first-time setup)
# This creates .venv, installs dependencies, and sets up pre-commit hooks
make init

# Activate the virtual environment
source .venv/bin/activate
```

### Running Tests

```shell
# Run all test suites (unit + e2e + cicd_utils)
uvx tox -m tests

# Run a specific test suite
uvx tox -e tests-unit        # Unit tests with coverage
uvx tox -e tests-e2e         # End-to-end tests
uvx tox -e tests-cicd_utils  # CI/CD utilities tests

# Run pytest directly with custom options
uvx tox -e pytest -- tests/unit/test_init.py --no-cov
uvx tox -e pytest -- -k "test_specific_function" --no-cov
```

### Linting and Formatting

```shell
# Run the main static checks
uvx tox -m static-quick

# Run the entire suite of static checks (incl. all pre-commit hooks)
# If running from the main branch, you'll need
# to skip the 'no-commit-to-branch' check with:
SKIP='no-commit-to-branch' uvx tox -m static

# Run all pre-commit hooks on all files
uvx pre-commit run --all-files

# Run specific pre-commit hooks
uvx pre-commit run ruff-format --all-files

# Run type checking with pyright only
uvx tox -e typing
```

### Documentation

```shell
# Build static documentation
uvx tox -e docs-static
```

## Project Map

```text
src/ridgeplot/
├── __init__.py           # Public API exports
├── _ridgeplot.py         # Main ridgeplot() function
├── _figure_factory.py    # Plotly Figure construction
├── _kde.py               # Kernel Density Estimation
├── _hist.py              # Histogram binning
├── _types.py             # Type aliases and type guards
├── _utils.py             # Utility functions
├── _missing.py           # Sentinel for missing values
├── _version.py           # Version string (setuptools-scm)
├── _color/               # Color handling
│   ├── colorscale.py     # Colorscale resolution
│   ├── css_colors.py     # CSS color parsing
│   ├── interpolation.py  # Color interpolation
│   └── utils.py          # Color utilities
├── _obj/traces/          # Trace objects
│   ├── area.py           # Area trace (filled curves)
│   ├── bar.py            # Bar trace (histograms)
│   └── base.py           # Base trace class
├── _vendor/              # Vendored dependencies
└── datasets/             # Built-in datasets
    └── data/             # CSV data files

tests/
├── conftest.py           # Shared pytest fixtures
├── unit/                 # Unit tests for individual modules
├── e2e/                  # End-to-end tests with expected outputs
│   └── artifacts/        # JSON artifacts for e2e comparisons
└── cicd_utils/           # Tests for CI/CD utilities

cicd_utils/               # CI/CD helper modules
├── cicd/                 # Scripts and test helpers
└── ridgeplot_examples/   # Example implementations for docs/testing
```

## Public API

The public API exposes one main function: `ridgeplot.ridgeplot(...) -> go.Figure`

The `ridgeplot()` docstring in `src/ridgeplot/_ridgeplot.py` contains the API contract.

`ridgeplot.datasets.load_probly()` and
  `ridgeplot.datasets.load_lincoln_weather()` are legacy loaders kept for
  backwards compatibility only.

## Key Data Flow

1. Users call `ridgeplot(samples=...)` or `ridgeplot(densities=...)`.
2. If samples are provided, KDE in `_kde.py` or histogram binning in `_hist.py`
   produces densities.
3. Densities are normalised if the `norm` parameter is set.
4. `create_ridgeplot()` in `_figure_factory.py` builds the Plotly Figure by
   resolving colors, creating traces, and applying layout settings.
5. The function returns a `plotly.graph_objects.Figure`.

## Key Files to Know

| File                               | Purpose                     |
|------------------------------------|-----------------------------|
| `src/ridgeplot/_ridgeplot.py`      | Main `ridgeplot()` function |
| `src/ridgeplot/_figure_factory.py` | Figure construction logic   |
| `src/ridgeplot/_types.py`          | All type aliases and guards |
| `tests/unit/test_ridgeplot.py`     | Core function tests         |
| `cicd_utils/ridgeplot_examples/`   | Example scripts for docs    |
| `docs/`                            | User and developer docs     |
| `tox.ini`                          | CI environment definitions  |
| `ruff.toml`                        | Linting configuration       |

Documentation: https://ridgeplot.readthedocs.io/en/stable/

## Workflow Expectations

### When Adding New Features

1. Start with `src/ridgeplot/_ridgeplot.py` to understand the entry point.
2. Add parameters following existing patterns and deprecation handling.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tpvasconcelos/ridgeplot](https://github.com/tpvasconcelos/ridgeplot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
