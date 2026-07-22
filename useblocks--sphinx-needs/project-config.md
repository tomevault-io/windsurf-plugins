---
trigger: always_on
description: This file provides guidance for AI coding agents working on the **Sphinx-Needs** repository.
---

# AGENTS.md

This file provides guidance for AI coding agents working on the **Sphinx-Needs** repository.

## Project Overview

Sphinx-Needs is a Sphinx extension for managing needs/requirements and specifications in documentation. It allows definition, linking, and filtering of class-like need-objects (requirements, specifications, implementations, test cases, etc.).

## Repository Structure

```
pyproject.toml        # Project configuration and dependencies

sphinx_needs/          # Main source code
├── api/               # Public API
├── directives/        # Sphinx directives (need, needtable, needflow, etc.)
├── functions/         # Dynamic functions
├── roles/             # Sphinx roles
├── services/          # External services (GitHub)
├── nodes/             # Docutils nodes
└── ...

tests/                 # Test suite
├── doc_test/          # Test documentation projects
├── js_test/           # Cypress JavaScript tests
├── benchmarks/        # Performance benchmarks
├── __snapshots__/     # Syrupy snapshot files
└── ...

docs/                  # Documentation source (RST files)
```

## Development Commands

All commands should be run via [`tox`](https://tox.wiki) for consistency. The project uses `tox-uv` for faster environment creation.

### Testing

```bash
# Run all tests (excluding benchmarks)
tox -- tests/

# Run a specific test file
tox -- tests/test_basic_doc.py

# Run a specific test function
tox -- tests/test_basic_doc.py::test_function_name

# Run tests with a specific Python version
tox -e py312 -- tests/

# Update snapshot tests (syrupy)
tox -- tests/ --snapshot-update

# Run benchmark tests
tox -e py312-benchmark -- tests/benchmarks/
```

### Documentation

```bash
# Build docs with furo theme (clean build)
CLEAN=true tox -e docs-furo

# Rebuild docs (incremental)
tox -e docs-furo

# Build with different themes
tox -e docs-alabaster
tox -e docs-rtd
tox -e docs-pds
tox -e docs-im

# Check documentation links
BUILDER=linkcheck tox -e docs-furo
```

### Code Quality

```bash
# Type checking with mypy
tox -e mypy

# Linting with ruff (auto-fix enabled)
tox -e ruff-check

# Formatting with ruff
tox -e ruff-fmt

# Run pre-commit hooks on all files
pre-commit run --all-files
```

## Code Style Guidelines

- **Formatter/Linter**: Ruff (configured in `pyproject.toml`)
- **Type Checking**: Mypy with strict mode (configured in `pyproject.toml`)
- **Docstrings**: Use Sphinx-style docstrings (not Google or NumPy style)
- **Function Signatures**: Use `/` and `*` to enforce positional-only and keyword-only arguments where appropriate

### Best Practices

- **Type annotations**: Use strict typing throughout. All function signatures should have complete type annotations. Use `TypedDict` for structured dictionaries, `Protocol` for duck typing.
- **Immutability**: Prefer immutable data structures. Needs are "sealed" (made immutable) after post-processing to prevent accidental modification.
- **Pure functions**: Where possible, write pure functions without side effects. This makes code easier to test and reason about.
- **Avoid global state**: Pass dependencies explicitly rather than relying on global variables.
- **Error handling**: Use specific exception types from `sphinx_needs.errors`. Provide helpful error messages with context.
- **Logging**: Use `sphinx_needs.logging.get_logger(__name__)` for logging. Use `log_warning()` for user-facing warnings.

### Docstrings

The Sphinx-style docstring format should be used throughout the codebase.
Types are not required in docstrings as they should be handled by type hints.

Example:

```python
def process_need(
    need_id: str,
    /,
    *,
    status: str | None = None,
) -> dict[str, str]:
    """Process a need and return its data.

    :param need_id: The unique identifier of the need.
    :param status: Optional status filter.
    :return: Dictionary containing need data.
    :raises ValueError: If need_id is not found.
    """
    ...
```

## Testing Guidelines

### Test Structure

- Tests use `pytest` with fixtures defined in `tests/conftest.py`
- Test documentation projects are in `tests/doc_test/`
- Snapshot testing uses `syrupy` - update snapshots with `--snapshot-update`
- JavaScript tests use Cypress and require the `@pytest.mark.jstest` marker

### Writing Tests

1. Create a test documentation project in `tests/doc_test/` if needed
2. Use the `test_app` fixture for Sphinx application testing
3. Mark tests appropriately:
   - `@pytest.mark.jstest` - JavaScript/Cypress tests
   - `@pytest.mark.benchmark` - Performance benchmarks
   - `@pytest.mark.fixture_file` - Tests using fixture files

### Test Best Practices

- **Test coverage**: Write tests for all new functionality and bug fixes
- **Isolation**: Each test should be independent and not rely on state from other tests
- **Descriptive names**: Test function names should describe what is being tested and expected outcome
- **Snapshot testing**: Use syrupy snapshots for complex HTML/output comparisons; review snapshot changes carefully
- **Edge cases**: Test error conditions, empty inputs, and boundary cases
- **Parametrization**: Use `@pytest.mark.parametrize` to test multiple scenarios with the same test logic

### Example Test Pattern

```python

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [useblocks/sphinx-needs](https://github.com/useblocks/sphinx-needs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
