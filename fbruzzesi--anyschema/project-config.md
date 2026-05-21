---
trigger: always_on
description: This file provides guidance for AI coding agents working on the `anyschema` project.
---

# AGENTS.md

This file provides guidance for AI coding agents working on the `anyschema` project.

## Project Overview

`anyschema` converts type specifications (Pydantic models, SQLAlchemy tables, TypedDict, dataclasses, attrs classes,
or plain Python dicts) to dataframe schemas (PyArrow, Polars, Pandas) using Narwhals as an intermediate representation.

## Key architectural principles

1. **Pipeline architecture:** Spec adapters normalize inputs → Parser pipeline converts types to Narwhals dtypes →
    Output format
2. **Dependency isolation:** Core codebase in `anyschema/` must be dependency-free except for:
    - Core dependencies: `narwhals` and `typing_extensions` (can always imported directly)
    - Library-specific files: `anyschema/parsers/{pydantic,attrs,sqlalchemy}.py` (never imported directly;
        loaded conditionally via `_dependencies.py`)
3. **Lazy imports:** Optional dependencies (Pydantic, attrs, SQLAlchemy) are checked via
    `anyschema/_dependencies.py` and only imported when available
4. **Parser order matters**: `ForwardRefStep` → `UnionTypeStep` → `AnnotatedStep` → `AnnotatedTypesStep` →
    `PydanticTypeStep` → `PyTypeStep`

Read `docs/architecture.md` for complete details on the pipeline design.

## Essential Commands

All commands use `uv` in the **active environment** (no virtual environment creation). Always run from repository root.

### Testing

```bash
# Run full test suite with coverage (requires >90% coverage)
uv run --active --no-sync --group tests pytest tests --cov=anyschema --cov=tests --cov-fail-under=90

# Run tests for a specific module
uv run --active --no-sync --group tests pytest tests/parsers/

# Run a specific test file
uv run --active --no-sync --group tests pytest tests/parsers/_builtin_test.py

# Run a specific test function
uv run --active --no-sync --group tests pytest tests/parsers/_builtin_test.py::test_specific_function

# Run doctests in source code
uv run --active --no-sync --group tests pytest anyschema --doctest-modules

# Run tests with verbose output
uv run --active --no-sync --group tests pytest tests -v

# Run tests and stop at first failure
uv run --active --no-sync --group tests pytest tests -x
```

### Linting and Formatting

```bash
# Run all linting and formatting (auto-fixes issues)
make lint

# Or run individual commands:
uvx ruff format anyschema tests
uvx ruff check anyschema tests --fix
uv tool run rumdl check .  # Markdown linting
```

**Important**: Always run `make lint` before committing. The project uses ruff with strict settings.

### Type Checking

```bash
# Run all type checkers
make typing

# Or run individually:
uv run --active --no-sync --group typing pyright anyschema
uv run --active --no-sync --group typing mypy anyschema
```

**Note**: Only type-check the `anyschema/` directory, not `tests/`.

### Documentation

```bash
# Serve docs locally with hot-reload (watches anyschema/ and docs/)
make docs-serve

# Build docs (strict mode - fails on warnings)
make docs-build
```

Documentation uses MkDocs with mkdocstrings for API reference generation.

## Code Style and Conventions

### Python Style

- **Python version**: Minimum 3.10, test on 3.10-3.14
- **Formatting**: Ruff (120 line length, Google docstring convention)
- **Imports**: Every file must start with `from __future__ import annotations`
- **Type hints**: Fully typed codebase (`disallow_untyped_defs = true`)
- **Docstrings**: Google style, comprehensive for all public APIs

### Testing Conventions

**Critical**: Use `pytest` with **function-based tests**, NOT `unittest` classes.

```python
# ✓ GOOD - Function-based pytest
def test_pydantic_adapter_basic():
    """Test that pydantic_adapter correctly extracts fields."""
    # Test implementation
    assert result == expected


# ✗ BAD - unittest classes
class TestPydanticAdapter(unittest.TestCase):
    def test_basic(self):
        self.assertEqual(result, expected)
```

**Testing best practices:**

- Use descriptive test names: `test_<feature>_<scenario>_<expected_outcome>`
- Use fixtures from `tests/conftest.py` (e.g., `auto_pipeline`, `pydantic_student_cls`)
- Parametrize tests for multiple scenarios: `@pytest.mark.parametrize(...)`
- Test both success and error cases
- Coverage requirement: >90% overall, >95% for non-excluded files
- Keep test files alongside corresponding source files (e.g., `tests/parsers/_builtin_test.py` for
    `anyschema/parsers/_builtin.py`)

### Dependency Management

**Core principle**: Keep `anyschema/` dependency-free except for specific patterns.

1. **Always allowed** (import directly):
    - `narwhals` (import as `nw`)
    - `typing_extensions`
    - Standard library

2. **Library-specific files** (can import their library):
    - `anyschema/parsers/pydantic.py` can import `pydantic`
    - `anyschema/parsers/attrs.py` can import `attrs`
    - `anyschema/parsers/sqlalchemy.py` can import `sqlalchemy`

3. **Conditional checks** (use `_dependencies.py`):

    ```python
    # ✓ GOOD - Check before importing
    from anyschema._dependencies import PYDANTIC_AVAILABLE, is_pydantic_base_model

    if PYDANTIC_AVAILABLE:
        from anyschema.parsers.pydantic import PydanticTypeStep

    # ✗ BAD - Direct import in main code

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FBruzzesi/anyschema](https://github.com/FBruzzesi/anyschema) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
