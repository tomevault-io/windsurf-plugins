---
trigger: always_on
description: This file provides guidance to AI agents (including Claude Code and other LLM-powered tools) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI agents (including Claude Code and other LLM-powered tools) when working with code in this repository.

## CRITICAL REQUIREMENTS

### Test Success
- ALL tests MUST pass for code to be considered complete and working
- Never describe code as "working as expected" if there are ANY failing tests
- Even if specific feature tests pass, failing tests elsewhere indicate broken functionality
- Changes that break existing tests must be fixed before considering implementation complete
- A successful implementation must pass linting, type checking, AND all existing tests

## Project Overview

gp-libs (this repository) packages the **cihai** Python library, which offers programmatic access to CJK (Chinese, Japanese, Korean) data built on top of `unihan-etl` and SQLAlchemy.

Key features:
- `Cihai` application object bootstraps and manages the UNIHAN dataset
- Dataset and plugin system (`extend.py`) for adding data sources or processors
- Config templating that respects XDG base directories
- SQLAlchemy-backed storage with convenience helpers in `db.py`
- Utility helpers for conversions, constants, and logging

## Development Environment

This project uses:
- Python 3.10+
- [uv](https://github.com/astral-sh/uv) for dependency management
- [ruff](https://github.com/astral-sh/ruff) for linting and formatting
- [mypy](https://github.com/python/mypy) for type checking
- [pytest](https://docs.pytest.org/) with doctests enabled
- [Sphinx](https://www.sphinx-doc.org/) + [Furo](https://github.com/pradyunsg/furo) for docs
- [gp-libs](https://gp-libs.git-pull.com) for shared docs/testing helpers

## Common Commands

### Setting Up Environment

```bash
# Install dependencies
uv pip install --editable .
uv pip sync

# Install with development dependencies (docs, tests, lint)
uv pip install --editable . -G dev
```

### Running Tests

```bash
# Run all tests (doctests included via pytest config)
just test
# or directly
uv run py.test

# Run a single test file
uv run py.test tests/test_core.py

# Run a specific test
uv run py.test tests/test_core.py::test_bootstrap

# Run tests with test watcher
just start
# or
uv run ptw .

# Run tests with doctests emphasized
uv run ptw . --now --doctest-modules
```

### Linting and Type Checking

```bash
# Run ruff for linting
just ruff
# or directly
uv run ruff check .

# Format code with ruff
just ruff-format
# or directly
uv run ruff format .

# Run mypy for type checking
just mypy
# or directly
uv run mypy src tests docs

# Watch mode for linting (using entr)
just watch-ruff
just watch-mypy
```

### Documentation

```bash
# Build documentation
just build-docs

# Start documentation server with auto-reload
just start-docs

# Live dev loop for docs
just dev-docs

# Update documentation CSS/JS
just design-docs
```

### Development Workflow

Follow this workflow for code changes:

1. **Format First**: `uv run ruff format .`
2. **Run Tests**: `uv run py.test`
3. **Run Linting**: `uv run ruff check . --fix --show-fixes`
4. **Check Types**: `uv run mypy`
5. **Verify Tests Again**: `uv run py.test`

## Code Architecture

cihai follows a dataset-centric design that bootstraps the UNIHAN corpus into a local SQL database and exposes a high-level API for lookups and extensions:

1. **Core** (`src/cihai/core.py`)
   - `Cihai` application object; merges config, bootstraps datasets/plugins, exposes `sql` handle.
   - Config templating via `expand_config` and XDG-aware directories.

2. **Database** (`src/cihai/db.py`)
   - SQLAlchemy engine/session setup and helpers for dataset storage.

3. **Extend** (`src/cihai/extend.py`)
   - Base classes for datasets/plugins, including SQLAlchemy mixins for shared DB access.

4. **Config & Internals** (`src/cihai/config.py`, `src/cihai/_internal/config_reader.py`)
   - Config loading/expansion from dict or YAML/JSON files; validation helpers.

5. **Data & Constants** (`src/cihai/constants.py`, `src/cihai/data/`)
   - Default configs, XDG paths, and dataset declarations for UNIHAN.

6. **Utilities** (`src/cihai/utils.py`, `src/cihai/conversion.py`, `src/cihai/log.py`, `src/cihai/types.py`)
   - Import helpers, conversion helpers, logging setup, and shared typing aliases.

7. **Exceptions** (`src/cihai/exc.py`)
   - Project-specific exception hierarchy.

## Testing Strategy

cihai uses pytest with doctests enabled via `pyproject.toml`. Key points:

- Root `conftest.py` provides fixtures that isolate `HOME`, working directory, and doctest namespace. Prefer these fixtures (`set_home`, `tmp_path`, etc.) over manual env manipulation.
- Doctests run automatically; keep examples minimal or move heavier scenarios into `tests/` or `examples/` with pytest fixtures.
- Use `pytest-watcher` (`uv run ptw .`) for continuous feedback during development.
- Favor `tmp_path` and `monkeypatch` fixtures instead of `tempfile`/`unittest.mock`.
- **Use functional tests only**: Write tests as standalone functions (`test_*`), not classes. Avoid `class TestFoo:` groupings. This applies to pytest tests, not doctests.

## Coding Standards


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cihai/cihai](https://github.com/cihai/cihai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
