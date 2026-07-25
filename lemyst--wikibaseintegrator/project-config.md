---
trigger: always_on
description: **WikibaseIntegrator** is a Python library for programmatically reading from and writing to Wikibase instances (like Wikidata). This is a complete rewrite of WikidataIntegrator with an object-oriented architecture supporting Items, Properties, Lexemes, and MediaInfo entities.
---

# WikibaseIntegrator Copilot Instructions

## Repository Overview

**WikibaseIntegrator** is a Python library for programmatically reading from and writing to Wikibase instances (like Wikidata). This is a complete rewrite of WikidataIntegrator with an object-oriented architecture supporting Items, Properties, Lexemes, and MediaInfo entities.

### High-Level Details

- **Project Type**: Python library package
- **Languages**: Python 3.10+ (tested up to Python 3.15-dev)
- **Package Manager**: Poetry (pyproject.toml-based)
- **Framework**: Object-oriented design with entity-specific classes
- **Target Runtime**: Python 3.10-3.14 in production
- **Repository Size**: ~1000+ Python files across core library and tests
- **Key Features**: Two execution modes (normal and "fast run"), comprehensive data type support, OAuth authentication, SPARQL query support

### Dependencies and Build System

The project uses **Poetry** for dependency management. **Always run `python -m poetry install --with dev` before any development work** to ensure all dependencies are available in the virtual environment.

Key dependency groups:
- **Main**: backoff, mwoauth, oauthlib, requests, requests-oauthlib, ujson
- **Dev**: pytest, pylint, mypy, codespell, flynt, pylint-exit  
- **Docs**: sphinx, sphinx-rtd-theme, sphinx-autodoc-typehints, toml
- **Coverage**: pytest-cov
- **Notebooks**: jupyter, jupyterlab

## Build, Test, and Validation Commands

**CRITICAL**: All commands must be run with `python -m poetry run` prefix to use the correct virtual environment.

### Environment Setup (REQUIRED FIRST)
```bash
# Install Poetry (if not available)
python -m pip install poetry

# Install all dependencies (ALWAYS run this first)
python -m poetry install --with dev

# For documentation work, add docs dependencies
python -m poetry install --with dev,docs

# For test coverage, add coverage dependencies  
python -m poetry install --with dev,coverage
```

### Testing Commands

```bash
# Run all tests (network tests may fail in restricted environments)
python -m poetry run pytest

# Run specific test file
python -m poetry run pytest test/test_wbi_core.py

# Run tests with verbose output
python -m poetry run pytest -v

# Run tests that don't require network (safer for CI)
python -m poetry run pytest test/test_datatype_time.py test/test_wbi_exceptions.py

# Check what tests are available
python -m poetry run pytest --collect-only
```

**Test Execution Notes:**
- Some tests require network connectivity (httpbin.org, wikidata.org)
- Tests typically complete in 0.1-3 seconds for individual modules
- Network-dependent tests may timeout or fail in restricted environments
- The test suite includes 78+ tests across multiple modules

### Code Quality and Linting

**Run all linting commands in this exact order** as used in CI:

```bash
# 1. Import sorting check (fast: <1s)
python -m poetry run isort --check --diff wikibaseintegrator test

# 2. Type checking (first run installs type stubs, ~30s; subsequent ~5s)
python -m poetry run mypy --install-types --non-interactive

# 3. Code linting (slowest: ~10s, has many warnings but doesn't fail build)
python -m poetry run pylint wikibaseintegrator test || python -m poetry run pylint-exit $?

# 4. Spell checking (fast: <1s)
python -m poetry run codespell wikibaseintegrator test

# 5. String formatting check (fast: <1s)
python -m poetry run flynt -f wikibaseintegrator test
```

**Linting Notes:**
- pylint typically shows 8.79/10 rating with many warnings (mostly style-related)
- pylint uses custom configuration in pyproject.toml (max-line-length: 180)
- Use `pylint-exit` to prevent pylint warnings from failing builds
- mypy requires initial type stub installation on first run
- All linting tools are configured via pyproject.toml

### Documentation Building

```bash
# Build HTML documentation (requires --with docs dependencies)
cd docs
python -m poetry run sphinx-build -b html source build/html

# API documentation regeneration
python -m poetry run sphinx-apidoc -e -f -o docs/source ./wikibaseintegrator/ -t docs/source/_templates
```

**Documentation Notes:**
- Documentation builds in ~30s with some warnings
- Built docs available at `docs/build/html/index.html`
- ReadTheDocs integration configured via `.readthedocs.yaml`
- Uses Sphinx with RTD theme and autodoc for API reference

### Package Installation and Import Testing

```bash
# Test package imports correctly
python -m poetry run python -c "import wikibaseintegrator; print('Import successful')"

# Install package in development mode (done automatically by Poetry)
# Poetry handles this via pyproject.toml [tool.poetry.dependencies]
```

## Project Layout and Architecture

### Core Architecture

```
wikibaseintegrator/
├── __init__.py                  # Main library entry point
├── wikibaseintegrator.py        # Primary WikibaseIntegrator class
├── entities/                    # Entity-specific classes
│   ├── baseentity.py           # Base class for all entities
│   ├── item.py                 # Item entity (Q-items)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LeMyst/WikibaseIntegrator](https://github.com/LeMyst/WikibaseIntegrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
