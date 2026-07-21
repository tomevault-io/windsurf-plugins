---
trigger: always_on
description: This file provides context for LLM assistants (Claude Code and similar tools) working in this repository.
---

# LLM Assistant Guide for `market-analy` package
This file provides context for LLM assistants (Claude Code and similar tools) working in this repository.

In all context files, a '@' prefixing a path indicates that the path is defined relative to the project root in which this `AGENTS.md` file is located.

## Skills

Identify all available skills in the @.agents\skills directory

## LLM context

Add the 'agents' label to any PR that amends:
- this @AGENT.md
- any SKILL.md file

## Project Overview

**market_analy** is a Python package for interactive charting and analysis of financial instruments. It provides GUIs built on bqplot, ipywidgets, and ipyvuetify for use in JupyterLab.
- Note that only a JupyterLab dark theme is currently supported (there is no support for a light theme).

See @pyproject.toml for project metadata and dependencies.

### Repository Layout

```
.agents/                       # instructions for LLM coding agents
├── skills/                    # skills for LLM coding agents
│   ├── create-pr/
│   │   └── SKILL.md
│   ├── dependencies-management/
│   │   └── SKILL.md
│   └── update-agents-md/
│       └── SKILL.md
.github/
├── workflows/
│   ├── build-test.yml
│   ├── draft-release-notes.yml
│   └── release.yml
└── release-drafter.yml
docs/
└── splash.png
src/
└── market_analy/                # Main package
    ├── trends/                  # Trend analysis subpackage
    │   ├── analy.py             # Trend analysis classes
    │   ├── charts.py            # Trend charting components
    │   ├── guis.py              # Trend GUI components
    │   └── movements.py         # Trend movement classes
    ├── utils/
    │   ├── bq_utils.py
    │   ├── dict_utils.py
    │   ├── ipyvuetify_utils.py
    │   ├── ipywidgets_utils.py
    │   ├── list_utils.py
    │   ├── maths_utils.py
    │   ├── mkt_prices_utils.py
    │   └── pandas_utils.py
    ├── analysis.py              # Core `Analysis` and `Compare` classes
    ├── cases.py                 # Base classes for displaying analyses over charts
    ├── charts.py                # bqplot figure creation
    ├── config.py                # Configuration constants
    ├── formatters.py            # Formatter functions and mappings
    ├── gui_parts.py             # GUI building blocks
    ├── guis.py                  # Interactive GUI components
    ├── standalone.py            # Standalone analysis functions
    └── trends_alt.py            # Deprecated; legacy trend analysis interface
tests/
├── resources/
├── tests_utils/                # Tests for `utils` subpackage
│   ├── __init__.py
│   ├── test_ipywidgets.py
│   ├── test_list.py
│   └── test_mkt_prices.py
├── __init__.py
├── conftest.py
├── test_analysis.py
├── test_charts.py
├── test_guis.py
├── test_standalone.py
├── test_trends.py
└── test_trends_alt.py
.pre-commit-config.yaml
.python-version
AGENTS.md
CLAUDE.md
LICENSE.txt
MANIFEST.in
README.md
mypy.ini
pyproject.toml
pytest.ini
requirements.txt
ruff.toml
uv.lock
```

## Technology Stack

| Category | Tools |
|---|---|
| Python | 3.10–3.14 (`.python-version` pins 3.14) |
| Package manager | `uv` |
| Build backend | `setuptools` + `setuptools_scm` |
| Testing | `pytest` |
| Linting/formatting | `ruff` |
| Type checking | `mypy` |
| Git hooks | `pre-commit` |
| Data Manipulation | `pandas`, `numpy` |
| Charting | `bqplot` |
| GUI Widgets | `ipywidgets`, `ipyvuetify` |
| Price Data | `market-prices` |
| Calendars of Market Hours | `exchange-calendars` |

The current project version is managed by `setuptools_scm` and written to `src/market_analy/_version.py`.
IMPORTANT: `src/maket_analy/_version.py` is auto-generated and you should not edit it.

## Development Workflows

### Setup

```bash
# Install dependencies using uv
uv sync

# Install pre-commit hooks
pre-commit install
```

### Testing

- test with `pytest`
- see @pytest.ini for configuration; options are applied automatically via `addopts`.
- shared fixtures are in @tests/conftest.py
- tests are in @tests/
- doctests are included to some methods/functions

Commands to run tests:
```bash
# All tests (including doctests under src/market_analy/)
pytest

# Tests in specific file
pytest tests/test_module.py

# Specific test
pytest tests/test_module.py::test_name

# With verbose output
pytest -v
```

### Pre-commit Hooks

See @.pre-commit-config.yaml for pre-commit implementation.

Pre-commit runs automatically on `git commit`.

To run manually:
```bash
pre-commit run --all-files
```

---

### Continuous Integration

GitHub Actions is used for CI. Defined workflows include:
- @.github/workflows/build-test.yml - runs full test suite on matrix of platforms and python versions.
- @.github/workflows/release.yml - releases a new version to PyPI.

## Code Conventions

### Architecture

The project employs a hierarchal class structure although compositional elements can be used if considered beneficial.

### Formatting

- format to `ruff` (Black compatible).  
- see @ruff.toml for configuration.

```bash
# Format code
ruff format .
```

### Linting

- lint with `ruff`.
- See lint sections of @ruff.toml for configuration (includes excluded files).
- type check with `mypy`.

```bash
# Check lint issues
ruff check .

# Type checking

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maread99/market_analy](https://github.com/maread99/market_analy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
