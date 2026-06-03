---
trigger: always_on
description: This file is for agentic coding agents working in `myskoda`.
---

## Purpose

This file is for agentic coding agents working in `myskoda`.
It documents the repository's real development workflow, code style, and guardrails.
Prefer small, targeted changes that fit existing patterns.

## Project Overview

- Language: Python, with the exact minimum version specified in `pyproject.toml`
- Package manager and environment tool: `uv`
- Dependencies, build backend, etc are specified in `pyproject.toml`
- Main package: `myskoda/`
- Tests: `pytest`
- Lint and formatting: `ruff` and `ruff format`
- Type checking: `pyright`
- Commit hooks: `pre-commit`

## Repository Layout

- `myskoda/`: library source
- `myskoda/auth/`: auth flow and token handling
- `myskoda/models/`: response models and enums
- `myskoda/cli/`: CLI entrypoints and helpers
- `tests/`: unit tests and fixtures
- `fixtures/`: additional fixture material
- `docs/`: documentation sources
- `scripts/`: helper scripts

## Setup

Use the repository's preferred workflow:

```bash
uv venv
source .venv/bin/activate
uv sync --all-extras
```

## Build, Lint, Typecheck, Test

Run commands from the repository root.

### Install dependencies

```bash
uv sync --all-extras
```

### Lint and format

```bash
uv run ruff check .
uv run ruff format .
```

### Type checking

```bash
uv run pyright
```

### Run pre-commit checks

This matches CI and local contributor guidance:

```bash
uv run pre-commit run --hook-stage manual --all-files
```

### Run the full test suite

Fast local run:

```bash
uv run pytest
```

CI-style run with coverage:

```bash
uv run pytest --cov-report term --cov-report xml:coverage.xml --cov=myskoda
```

### Run a single test file

```bash
uv run pytest tests/test_utils.py
```

### Run a single test function

```bash
uv run pytest tests/test_utils.py::test_async_debounce
```

### Run tests by keyword expression

```bash
uv run pytest -k debounce
```

### Run a single async-heavy test with logs visible

Pytest logging is already enabled in `pyproject.toml`, so this is often enough:

```bash
uv run pytest tests/test_rest_api.py::test_get_info
```

## Release and Packaging

- Build metadata is defined in `pyproject.toml`
- The package version is dynamic
- Do not hand-edit version strings unless the repository workflow changes

## CI Facts

The GitHub workflow in `.github/workflows/test.yaml` currently does this:

- `uv sync --all-extras`
- `uv pip install pre-commit`
- `uv run pre-commit run --hook-stage manual --all-files`
- `uv run pytest --cov-report term --cov-report xml:coverage.xml --cov=myskoda`

Prefer validating with the same commands before finishing substantial changes.

## Code Style

Follow the existing code, not generic Python advice.

### Formatting

- Line length is `100`
- Target runtime is Python `3.13`
- Use `ruff format` for formatting
- Keep code ASCII unless the file already requires Unicode
- Preserve existing blank-line and docstring patterns where practical

### Imports

- Keep imports grouped in the existing order: standard library, third-party, local package
- Use direct imports rather than lazy imports unless there is a concrete reason
- Prefer explicit imports over wildcard imports
- In-package imports use both absolute and relative style today; follow the surrounding file's pattern
- Avoid introducing unused imports just to satisfy speculative future work

### Typing

- Add type hints for new public functions, methods, and important internal helpers
- Use modern built-in generics like `list[str]`, `dict[str, Any]`, `tuple[int, ...]`
- Use union syntax like `Foo | None` instead of `Optional[Foo]`
- Use `collections.abc` types for callable and iterator contracts when appropriate
- The codebase already uses `ParamSpec`, `Protocol`, `Self`, and generic dataclasses; reuse those patterns when helpful
- Respect `pyright`; do not add ignore comments unless necessary and justified by surrounding code
- Match the file's existing annotation density rather than over-annotating trivial locals

### Naming

- Classes use `PascalCase`
- Functions, methods, variables, and modules use `snake_case`
- Constants use `UPPER_SNAKE_CASE`
- Enum members follow the style already present in each enum
- Test files are named `test_*.py`
- Test functions are named `test_*`
- Prefer descriptive domain names such as `authorization`, `charging_profile`, `vehicle_status`, `target_vin`

### Docstrings

- Ruff enforces pydocstyle with the Google convention
- Public modules, classes, and functions usually have docstrings
- Tests are exempt from docstring linting, but many tests still include useful docstrings
- Keep docstrings short and factual
- Do not add noisy comments that restate obvious code

### Data Models and Enums

- This project leans heavily on typed response models under `myskoda/models/`
- Reuse existing model classes and enums before adding new string literals or ad hoc dict parsing
- Prefer extending the existing model layer instead of returning partially structured dicts
- When API values are enumerated, prefer `StrEnum` or the existing enum abstractions already used in the package

### Async Conventions

- The library is async-first; many public APIs are `async def`
- Preserve async boundaries instead of wrapping async behavior in sync helpers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skodaconnect/myskoda](https://github.com/skodaconnect/myskoda) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
