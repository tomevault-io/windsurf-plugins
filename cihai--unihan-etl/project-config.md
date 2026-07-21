---
trigger: always_on
description: Guidance for AI agents (Claude Code, Cursor, et al.) working in this repository.
---

# AGENTS.md

Guidance for AI agents (Claude Code, Cursor, et al.) working in this repository.

## CRITICAL REQUIREMENTS

### Test Success
- ALL tests MUST pass before you describe changes as complete.
- Do not claim the code "works" while any test, lint, or type check fails.
- Fix regressions introduced by your changes; do not mark work finished with broken CI.
- Successful work means: formatting, linting, type checking, and test suites are all green.

## Project Overview

unihan-etl is a Python ETL tool that downloads the Unicode UNIHAN database, normalizes it, and exports it in multiple formats (CSV, JSON, YAML, or Python objects). It provides both a library API and a CLI for fetching, expanding, and pruning UNIHAN fields. The library is typed and ships pytest fixtures for quickly bootstrapping test datasets.

Key capabilities:
- Download, cache, and validate official UNIHAN releases.
- Convert UNIHAN text files into tabular or structured exports.
- Options to expand multi-value fields and prune empty values.
- Typed Python API plus CLI entry point for batch exports.
- Pytest plugin supplying ready-to-use quick/full datasets.

## Development Environment

This project uses:
- Python 3.10+
- [uv](https://github.com/astral-sh/uv) for dependency management
- [ruff](https://github.com/astral-sh/ruff) for linting/formatting
- [mypy](https://github.com/python/mypy) for type checking
- [pytest](https://docs.pytest.org/) for testing
- [gp-libs](https://gp-libs.git-pull.com) for shared dev tooling (docs/test helpers)

## Common Commands

### Setting Up Environment

```bash
# Install dependencies (editable)
uv pip install --editable .
uv pip sync

# Install with development extras
uv pip install --editable . -G dev
```

### Running Tests

```bash
# Run all tests
just test          # -> uv run py.test
# or directly
uv run py.test

# Watch tests
just start         # runs tests once then starts ptw
uv run ptw .

# Dead code scan
just vulture
```

### Linting and Type Checking

```bash
# Lint
just ruff
uv run ruff check .

# Format
just ruff-format
uv run ruff format .

# Lint with fixes
uv run ruff check . --fix --show-fixes

# Type check
just mypy
uv run mypy src tests
```

### Documentation

```bash
just build-docs     # build Sphinx docs
just start-docs     # autobuild + serve
just design-docs    # rebuild CSS/JS assets
```

## Code Architecture

1. **core.py** (`src/unihan_etl/core.py`)  
   - Houses the ETL pipeline and `Packager` class: download, extract, normalize, expand, prune, and export UNIHAN data.
   - CLI entry helpers (`from_cli`) and export writers (CSV/JSON/YAML/Python).
2. **options.py** (`src/unihan_etl/options.py`)  
   - Dataclass-like configuration object with defaults for paths, formats, caching, logging, and selected fields/files.
3. **constants.py** (`src/unihan_etl/constants.py`)  
   - Field lists, manifest mapping of UNIHAN files to fields, default paths, and allowed export formats.
4. **expansion.py** (`src/unihan_etl/expansion.py`)  
   - Expands multi-value UNIHAN fields into structured lists/dicts; handles delimiters and nested data.
5. **util.py** (`src/unihan_etl/util.py`)  
   - Helpers for progress display, field resolution, Unicode codepoint handling (`ucn_to_unicode`), and manifest utilities.
6. **types.py** (`src/unihan_etl/types.py`)  
   - Shared TypedDicts/TypeAliases for exported structures and options.
7. **pytest_plugin.py** (`src/unihan_etl/pytest_plugin.py`)  
   - Pytest fixtures for “quick” (small) and “full” (complete) UNIHAN datasets; manages cache locations and packagers.
8. **__main__.py / test.py**  
   - CLI entry point (`python -m unihan_etl`) and legacy test harness.

## Testing Strategy

- Prefer the provided pytest fixtures (`unihan_quick_*`, `unihan_full_*`) instead of ad-hoc downloads; they bootstrap caches and zip fixtures for deterministic tests.
- Quick fixtures create a small synthetic UNIHAN subset suitable for fast unit tests; full fixtures mirror the complete dataset and may be slower/heavier.
- Use `uv run ptw .` or `just start` for watch mode during development.
- When adding new fixtures, keep them under the existing cache paths defined in `pytest_plugin.py`.
- Avoid network access in unit tests; rely on cached zips or local fixtures.

### Testing Guidelines
- Favor `tmp_path` over manual tempfile handling.
- Use `monkeypatch` and fixtures before reaching for mocks.
- Add docstring examples to showcase CLI/library usage only when short and stable; move longer examples to dedicated test files under `tests/`.

## Coding Standards

- Always begin Python modules with `from __future__ import annotations`.
- Prefer namespace imports for stdlib (`import typing as t`, `import pathlib`, etc.); third-party packages may use `from X import Y`.
- Use NumPy-style docstrings in reStructuredText format for public APIs.
- Format before review: `uv run ruff format .`; then lint (`uv run ruff check . --fix --show-fixes`); then type-check (`uv run mypy`); re-run tests.
- Doctests: keep short, narrative examples in docstrings; move complex flows into `tests/examples/**`.
- Pytest: favor existing fixtures and `tmp_path`/`monkeypatch` instead of heavy mocks.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cihai/unihan-etl](https://github.com/cihai/unihan-etl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
