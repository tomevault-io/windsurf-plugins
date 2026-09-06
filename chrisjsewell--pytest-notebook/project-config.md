---
trigger: always_on
description: Provides the `%pytest` line and `%%pytest` cell magics (loaded via `%load_ext pytest_notebook.ipy_magic`) to run pytest against inline test content from within a notebook.
---

# AGENTS.md

This file provides guidance for AI coding agents working on the **pytest-notebook** repository.

## Project Overview

pytest-notebook is a [pytest](https://docs.pytest.org) plugin for regression-testing and executing [Jupyter Notebooks](https://jupyter.org/). It provides:

- Collection of `.ipynb` files as pytest tests (each notebook becomes a test item)
- Execution of notebooks via [nbclient](https://nbclient.readthedocs.io), with optional [coverage.py](https://coverage.readthedocs.io) integration
- Regression testing of notebook outputs by diffing against the stored notebook, using [nbdime](https://nbdime.readthedocs.io)
- Configurable diff ignoring, regex replacement, and output post-processing
- An `nb_regression` fixture and a `%%pytest` IPython magic for interactive use

pytest-notebook is designed to make notebooks a first-class, testable artifact, verifying that they execute cleanly and that their outputs remain stable.

Documentation is hosted at [pytest-notebook.readthedocs.io](https://pytest-notebook.readthedocs.io).

## Repository Structure

```
pyproject.toml          # Project configuration and dependencies (flit)
tox.ini                 # Tox environments + [pytest] self-testing config

pytest_notebook/        # Main source code
├── __init__.py         # Package init / version
├── plugin.py           # pytest hooks, `nb_regression` fixture, notebook collector
├── nb_regression.py    # NBRegressionFixture - core execute + diff logic
├── execution.py        # nbclient-based execution, coverage.py integration
├── diffing.py          # nbdime diffing, filtering, and formatting of diffs
├── notebook.py         # Notebook loading + `nbreg` metadata config (JSON-schema validated)
├── post_processors.py  # Entry-point post-processors (coalesce_streams, blacken_code, beautifulsoup)
├── normalizers.py      # Entry-point diff normalizers (strip_ansi, mask_timestamps, ...)
├── ipy_magic.py        # `%pytest` / `%%pytest` IPython magic
├── utils.py            # Utility helpers (e.g. autodoc)
├── resources/          # JSON schema and other package resources
└── example_nbs/        # Example notebooks

tests/                  # Test suite
├── conftest.py         # Shared fixtures
├── test_nb_regression.py   # NBRegressionFixture tests
├── test_execution.py       # Execution tests
├── test_nb_diff.py         # Notebook diffing tests
├── test_cell_diff.py       # Cell-level diffing tests
├── test_filter_diff.py     # Diff filtering tests
├── test_notebook.py        # Notebook loading / config tests
├── test_coalesce_streams.py, test_postprocessors/  # Post-processor tests
├── test_plugin_collector.py, test_plugin_fixture.py # Plugin/collector/fixture tests
├── test_ipy_magic.py       # IPython magic tests
├── test_utils.py
└── raw_files/          # Stored fixtures for pytest-regressions comparisons

docs/                   # Documentation source (Sphinx + myst-nb)
└── source/
    ├── conf.py         # Sphinx configuration
    ├── index.rst       # Documentation index
    ├── changelog.md    # Changelog (update on every user-facing change)
    ├── user_guide/     # User guide notebooks/pages
    ├── apidoc/         # API documentation
    └── literal_includes/
```

## Development Commands

Tests can be run either via [`tox`](https://tox.wiki) (recommended, for isolated environments) or directly with pytest in a local install.

### Testing

```bash
# Run the default environment
tox

# Run tests with a specific Python version
tox -e py311 -- {pytest args}

# Run a specific test file
tox -e py311 -- tests/test_nb_regression.py

# Run a specific test function
tox -e py311 -- tests/test_nb_regression.py::test_basic_execution
```

Or, for a local (non-tox) workflow, install the test dependency group and run pytest directly:

```bash
# Install the package with the PEP 735 `test` dependency group
pip install --group test -e .

# Run the tests
pytest
```

> Test dependencies live in the PEP 735 `[dependency-groups]` `test` group in `pyproject.toml`
> (this replaces the old `testing` extra). Documentation dependencies are the `docs` extra.

### Documentation

```bash
# Build docs (clean rebuild)
tox -e docs-clean

# Build docs (incremental update)
tox -e docs-update
```

The docs environment runs `make` in the `docs/` directory using the `docs` extra.

### Code Quality

```bash
# Run pre-commit hooks on all files (ruff + ruff-format)
pre-commit run --all-files

# Linting and formatting individually
pre-commit run ruff --all-files
pre-commit run ruff-format --all-files
```

There is no mypy / type-checking step configured in this repository.

## Code Style Guidelines

- **Formatter/Linter**: Ruff (configured in `pyproject.toml`, `[tool.ruff.lint]`)
- **Pre-commit**: Use pre-commit hooks for consistent code style (`.pre-commit-config.yaml`)
- **Python**: Requires Python `>=3.10`

### Best Practices

- **Docstrings**: Use Sphinx-style (`:param:` / `:return:`) docstrings, as used throughout the codebase.
- **attrs**: Configuration classes (e.g. `NBRegressionFixture`) use [attrs](https://www.attrs.org) with validators; follow the existing pattern when adding options.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chrisjsewell/pytest-notebook](https://github.com/chrisjsewell/pytest-notebook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
