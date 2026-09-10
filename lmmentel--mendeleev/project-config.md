---
trigger: always_on
description: **mendeleev** is a Pythonic periodic table of elements library that provides a convenient Python API for accessing various properties of elements, ions, and isotopes. It integrates with pandas for data access and offers visualization capabilities through bokeh, plotly, and seaborn.
---

# Mendeleev - Claude Code Assistant Guide

## Project Overview

**mendeleev** is a Pythonic periodic table of elements library that provides a convenient Python API for accessing various properties of elements, ions, and isotopes. It integrates with pandas for data access and offers visualization capabilities through bokeh, plotly, and seaborn.

- **Repository**: https://github.com/lmmentel/mendeleev
- **Documentation**: https://mendeleev.readthedocs.io
- **License**: MIT
- **Current Version**: 1.1.0

## Technology Stack

### Core Dependencies
- **Python**: 3.9 - 3.13
- **pandas**: Data manipulation and analysis (>=1.1.0)
- **SQLAlchemy**: Database ORM and data models (>=1.4.0)
- **pydantic**: Data validation and settings management (^2.9.2)
- **pint**: Unit conversions and physical quantities (^0.24.4)
- **numpy**: Numerical computing (^2.0)

### Development Tools
- **Poetry**: Dependency and environment management
- **pytest**: Testing framework (with pytest-xdist and pytest-cov)
- **Sphinx**: Documentation generation (with sphinx-material theme)
- **pre-commit**: Git hooks for code quality
- **ruff**: Linting and code formatting
- **alembic**: Database migrations
- **invoke**: Task automation

### Optional Visualization Dependencies
- **bokeh**: Interactive visualizations (^3.0)
- **plotly**: Interactive plots (^5.0)
- **seaborn**: Statistical visualizations (>=0.12)

## Project Structure

```
mendeleev/
├── mendeleev/           # Main package source code
│   ├── __init__.py
│   ├── models.py        # SQLAlchemy data models
│   ├── fetch.py         # Data fetching utilities
│   ├── db.py            # Database utilities
│   ├── econf.py         # Electronic configuration
│   ├── elements.db      # SQLite database with element data
│   ├── cli.py           # Command-line interface
│   ├── vis/             # Visualization modules
│   │   ├── bokeh.py
│   │   ├── plotly.py
│   │   ├── seaborn.py
│   │   └── utils.py
│   └── interfaces/      # External data interfaces
├── tests/               # Test suite
│   ├── test_element.py
│   ├── test_isotope.py
│   ├── test_ion.py
│   ├── test_fetch.py
│   ├── test_vis.py
│   └── test_econf/
├── docs/                # Sphinx documentation
│   ├── source/          # Documentation source files
│   │   ├── conf.py      # Sphinx configuration
│   │   ├── notebooks/   # Jupyter notebook tutorials
│   │   └── api/         # API documentation
│   ├── Makefile         # Documentation build commands
│   └── build/           # Generated documentation
├── notebooks/           # Development notebooks (not in docs)
├── alembic/             # Database migration scripts
│   └── versions/
├── tasks.py             # Invoke task definitions
├── pyproject.toml       # Poetry configuration and dependencies
└── CONTRIBUTING.md      # Contribution guidelines
```

## Development Setup

### Prerequisites
- Python 3.9 or higher
- [Poetry](https://python-poetry.org/) for dependency management
- Git

### Initial Setup

1. **Clone the repository**
   ```bash
   git clone https://github.com/lmmentel/mendeleev.git
   cd mendeleev
   ```

2. **Install dependencies**
   ```bash
   # Install core dependencies
   poetry install

   # Install with visualization dependencies
   poetry install --with vis
   ```

3. **Activate the virtual environment**
   ```bash
   poetry shell
   ```

## Running Tests

### Basic Test Execution

```bash
# Run all tests
poetry run pytest

# Run tests with coverage report
poetry run pytest --cov=mendeleev

# Run tests in parallel (using pytest-xdist)
poetry run pytest -n auto

# Run tests with duration reporting (slowest 10 tests)
poetry run pytest --durations=10
```

### Run Specific Tests

```bash
# Run tests in a specific file
poetry run pytest tests/test_element.py

# Run a specific test function
poetry run pytest tests/test_element.py::test_function_name

# Run tests matching a pattern
poetry run pytest -k "element"
```

### Test Configuration

Tests are configured in `pyproject.toml`:
```toml
[tool.pytest.ini_options]
minversion = "8.0"
addopts = "--durations=10 -n auto"
```

## Building Documentation

### Prerequisites

Documentation requires additional dependencies beyond the core package:

```bash
# Install documentation dependencies (one-time setup)
poetry run pip install -r docs/requirements.txt
```

Required packages include:
- sphinx, nbsphinx (documentation generator)
- sphinx-material (documentation theme)
- myst-parser (Markdown support)
- sphinxcontrib-bibtex (bibliography)
- bokeh, plotly, seaborn (for visualization examples)
- ipython, ipykernel (for notebook examples)

### Local Documentation Build

**Prerequisites:**

First, install documentation dependencies (one-time setup):
```bash
poetry run pip install -r docs/requirements.txt
```

This includes:
- `sphinx` - Documentation builder
- `sphinx-material` - Material theme
- `sphinx-design` - Grid and card layouts for modern UI
- `nbsphinx` - Jupyter notebook support
- `sphinxcontrib-bibtex` - Bibliography support
- `myst-parser` - Markdown support

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lmmentel/mendeleev](https://github.com/lmmentel/mendeleev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
