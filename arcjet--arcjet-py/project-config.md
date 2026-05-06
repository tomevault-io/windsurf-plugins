---
trigger: always_on
description: This document provides essential information for coding agents working on the
---

# AGENTS.md — Coding Agent Onboarding Guide

This document provides essential information for coding agents working on the
arcjet-py repository for the first time.

## Repository overview

**Project**: Arcjet Python SDK
**Purpose**: Security SDK for Python applications providing bot detection, rate
limiting, email validation, Shield WAF, and attack protection for Flask,
FastAPI, and other Python web frameworks.
**Language**: Python 3.10+
**Package manager**: uv

## Quick start

### Environment setup

1. **Python version**: 3.10+ (see `.python-version`)
2. **Install uv** (if not already installed):
   ```bash
   pip install uv
   ```
3. **Install dependencies**:
   ```bash
   uv sync
   ```
   This creates a virtual environment in `.venv` and installs all dependencies
   from `uv.lock`.

### Development workflow

Always prefix commands with `uv run`. Common tasks are wrapped in the
`Makefile`:

```bash
make check       # All lint + type + API-break checks (single command)
make format      # Auto-fix imports and format code
make test        # All tests (unit + integration + analyze, with coverage)
make bench       # Run benchmarks
```

## Build, test, and lint commands

### Testing

Run all tests with a single command:

```bash
uv run pytest
```

This runs both unit tests (in `tests/unit/`) and integration tests (in
`tests/fastapi/`, `tests/flask/`, etc.) together.

**Test organization**: Tests use pytest fixtures for protobuf mocking, allowing
all tests to run together without cross-contamination. See
`tests/TESTING_PATTERNS.md` for detailed testing conventions.

**Coverage**: Both test suites enforce an 80% minimum coverage threshold
(`fail_under = 80` in `pyproject.toml`). Generated code (protobuf and witgen
output) is excluded from coverage via `[tool.coverage.run] omit`.

WASM binding tests are in `tests/analyze/` and run as part of the main test suite.

### Linting, formatting, and type checking

```bash
uv run ruff check --select I --fix  # Sort imports
uv run ruff format                  # Format code
uv run ruff check                   # Lint
uv run ty check                     # ty type checker
uv run pyright                      # Pyright type checker
```

Both type checkers must pass. Some test files are excluded from type checking
(see `[tool.pyright]` and `[tool.ty.src]` in `pyproject.toml`).

Suppression comments:
- `# type: ignore[error-code]` — Recognized by both pyright and ty.
- `# pyright: ignore[rule]` — Pyright-specific.
- `# ty: ignore[rule]` — ty-specific (e.g., conditional stdlib imports on 3.10).

### API breaking change detection

```bash
uv run griffe check arcjet -s src --against origin/main
```

PRs with breaking changes must be labeled with `breaking` label to be merged.

**IMPORTANT**: Always run this check before committing changes. Breaking changes
must be avoided unless absolutely necessary. Whenever there is a breaking
change, existing code must not break — we must maintain backward compatibility
and provide clear migration paths. This can be docs, deprecation warnings, and
keeping the existing API surface intact with internal changes.

## Code organization

### Source structure

- `src/arcjet/` — Main SDK package. Public API in `__init__.py`, client in
  `_client.py`, rules in `_rules.py`, local WASM evaluation in `_local.py`.
  Protobuf code in `proto/` is **generated — do not edit**.
- `src/arcjet/_analyze/` — WASM component integration with typed Python bindings.
  See `docs/WITGEN.md` for binding generation and
  `docs/WASMTIME.md` for wasmtime-py details.
- `tools/witgen/` — WIT-to-Python code generator (configured by `witgen.toml`).
- `tests/` — Unit tests (`tests/unit/`), integration tests
  (`tests/fastapi/`, `tests/flask/`), WASM binding tests (`tests/analyze/`),
  and benchmarks (`tests/benchmarks/`). See `tests/TESTING_PATTERNS.md`.

## Coding conventions

### Python style

1. **Future imports**: Always use `from __future__ import annotations`.

2. **Python version**: >=3.10. Do not use features from 3.11+ (e.g.
   `typing.Self`, `ExceptionGroup`).

3. **Type hints**: Fully type-annotated codebase.
   - Use modern type syntax (e.g., `list[str]` not `List[str]`).
   - Use `Union` instead of `X | Y` in runtime-evaluated type aliases (the
     latter works in annotations with `from __future__ import annotations`
     but not in runtime-evaluated positions on 3.10).

4. **Dataclasses**: Prefer `@dataclass(frozen=True, slots=True)` for immutable
   data structures.

5. **Enums**: Use `str, Enum` pattern for string enums:
   ```python
   class Mode(str, Enum):
       DRY_RUN = "DRY_RUN"
       LIVE = "LIVE"
   ```

6. **Private modules**: Prefix with underscore (e.g., `_enums.py`,
   `_logging.py`) for internal-only modules.

### Framework support

The SDK is **framework-agnostic** with explicit support for ASGI (Starlette,
FastAPI), Flask/Werkzeug, and Django. The `_context.py` module provides
`coerce_request_context()` to convert framework requests to a common
`RequestContext` type.

### Environment variables

- `ARCJET_KEY` — API key (passed to `arcjet()` or `arcjet_sync()`)
- `ARCJET_ENV` — Set to `"development"` for development mode
- `ARCJET_LOG_LEVEL` — Log level (`debug`, `info`, `warning`, `error`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arcjet/arcjet-py](https://github.com/arcjet/arcjet-py) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
