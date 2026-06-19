---
trigger: always_on
description: This document provides essential guidance for AI agents working on this repository. It covers tooling, conventions, and workflows needed to contribute effectively.
---

# AI Agent Development Guide

This document provides essential guidance for AI agents working on this repository. It covers tooling, conventions, and workflows needed to contribute effectively.

## How to use this document

### When to read this file
- First time working on this repository
- Before making any code changes or commits
- When unsure about code conventions or tooling

## Before every commit
- Ensure all code has type annotations
- Add Google-style docstrings (NO types, NO leading articles)
- Pre-commit hooks will run automatically and must pass

## When in doubt
- Check code conventions section below
- Run individual tools to identify issues
- Ask user for clarification on ambiguous requirements

## Code conventions

### Docstrings

Required for: all public modules, classes, functions, and methods

#### Format: Google-style
1. Do not place type information in docstrings - use type annotations only
2. Do not use leading articles in parameter, return, and error descriptions "a", "an", or "the"

##### Section order
1. Args
2. Returns
3. Raises
4. Yields
5. Examples
6. Note
7. Warning

#### Example
```python
def process_data(input_data: list[str], threshold: int = 10) -> dict[str, int]:
    """Process input data and return summary statistics.

    Args:
        input_data: strings to process
        threshold: minimum count threshold for inclusion

    Returns:
        Mapping of categories to counts

    Examples:
        >>> process_data(["a", "b"], 5)
        {"valid": 2}
    """
```

#### Incorrect example (do not do this!)
```python
def process_data(input_data: list[str], threshold: int = 10) -> dict[str, int]:
    """Process input data and return summary statistics.

    Args:
        input_data (list[str]): A list of strings to process.  # ❌ Has type and article
        threshold (int): A minimum count threshold.            # ❌ Has type and article

    Returns:
        dict[str, int]: A dictionary mapping categories.       # ❌ Has type and article
    """
```

### Type annotations

Requirements
- All functions must have complete type annotations
- Use modern syntax: `list[str]`, `dict[str, int]` (not `List[str]`, `Dict[str, int]`)
- Use `|` for union types (Python 3.10+): `str | None`
- Import types from `typing` only when necessary (prefer built-ins)

#### Verification
```bash
uv run ty check .
```

### Code formatting

Via ruff
- Line length: 120
- Indentation: 4 spaces (no tabs except Makefiles)
- Encoding: UTF-8
- Line endings: LF (Unix-style)
- Trailing newline: required
- No trailing whitespace

### Naming conventions
- Functions/methods: snake_case
- Variables: snake_case
- Constants: UPPER_SNAKE_CASE
- Classes: PascalCase
- Modules: snake_case
- Private attributes/methods: _leading_underscore

### Imports
- Absolute imports preferred
- Group imports: standard library, third-party, local
- No wildcard imports (`from module import *`) except in `__init__.py`
- Import sorting handled by ruff (isort)

### Error handling
- Use specific exceptions (ValueError, OSError, etc.) rather than generic Exception
- Avoid bare `except:` clauses; catch specific exceptions
- Use context managers (`with` statements) for resource management
- Log errors appropriately using the `logging` module
- Raise custom exceptions for domain-specific errors

### General style
- Use f-strings for string formatting (Python 3.6+)
- Prefer list/dict comprehensions over loops when appropriate
- Use `pathlib.Path` for file operations instead of `os.path`
- Avoid global variables; use dependency injection
- Write readable code; prefer explicit over implicit

## Essential commands

```bash
# Setup
uv sync                         # Install dependencies

# Code quality
uv run ruff format .                # Format code
uv run ruff check .                 # Lint code
uv run ty check .                   # Type check

# Testing
uv run pytest                   # Run tests
uv run pytest --cov             # Run tests with coverage
uv run pytest -k "pattern"      # Run test matching pattern
uv run pytest -v                # Verbose output

# Git workflow
git add .
git commit -m "feat: message"   # Hooks run automatically

# Package management
uv add <package>                  # Add a package
uv add --dev <package>            # Add a package to dev
uv lock                           # Check the lockfile matches the pyproject.toml
uv sync --upgrade                 # Update packages from the lockfile
uv tree                           # Print the dependency tree
```

## Code quality tools

### Ruff (formatting & linting)

Enabled rule categories:
- `B` - bugbear (common bugs and design problems)
- `D` - pydocstyle (docstring conventions)
- `E`/`W` - pycodestyle (PEP 8 style errors and warnings)
- `F` - pyflakes (logical errors)
- `I` - isort (import sorting)
- `N` - pep8-naming (naming conventions)
- `C4` - comprehensions (list/dict/set comprehension improvements)
- `PL` - pylint (code quality and error detection)
- `PT` - pytest-style (pytest best practices)
- `PIE` - misc lints (miscellaneous improvements)
- `PYI` - flake8-pyi (stub file best practices)
- `TID` - tidy imports (import hygiene)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rowansci/openconf](https://github.com/rowansci/openconf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
