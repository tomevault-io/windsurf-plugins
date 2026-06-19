---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## AI Agent Development Guide

This document provides essential guidance for AI agents working on this repository. It covers tooling, conventions, and workflows needed to contribute effectively.

## How to use this document

### When to read this file
- First time working on this repository
- Before making any code changes or commits
- When unsure about code conventions or tooling

## Before every commit
- Ensure all code has type annotations
- Add sphinx-style docstrings (NO types, NO leading articles)
- Run checks: `prek -a`
- Pre-commit hooks will run automatically and must pass

## When in doubt
- Check code conventions section below
- Run individual tools to identify issues
- Ask user for clarification on ambiguous requirements

## Code conventions

### Docstrings

Required for: all public modules, classes, functions, and methods

#### Format: Sphinx-style
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
    """
    Process input data and return summary statistics.

    :param input_data: strings to process
    :param threshold: minimum count threshold for inclusion
    :return: Mapping of categories to counts

    >>> process_data(["a", "b"], 5)
    {"valid": 2}
    """
```

#### Incorrect example (do not do this!)
```python
def process_data(input_data: list[str], threshold: int = 10) -> dict[str, int]:
    """
    Process input data and return summary statistics.

    :param input_data: (list[str]): A list of strings to process.  # ❌ Has type and article
    :param threshold: (int): A minimum count threshold.            # ❌ Has type and article
    :return: dict[str, int]: A dictionary mapping categories.      # ❌ Has type and article
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
mypy .
```

### Code formatting

Via ruff
- Line length: 160
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

## Repository overview

Purpose: STructured JSON Atom/Molecule Encoding Scheme (stjames) - a Pydantic-based schema library for passing molecular and calculation data between components of the Rowan computational chemistry platform. Provides validation and intelligent default selection for computational chemistry settings.

Structure:
- `stjames/` - source code (flat layout)
- `stjames/workflows/` - high-level computational chemistry workflow definitions
- `stjames/data/` - element data and isotope information
- `stjames/atomium_stjames/` - PDB/mmCIF parsing utilities
- `tests/` - test suite
- `.github/workflows/` - CI/CD configuration

Python Version: >=3.11

Key configuration files:
- `pyproject.toml` - Project metadata, dependencies, all tool configuration
- `.pre-commit-config.yaml` - Prek hook configuration
- `.coveragerc` - Test coverage settings
- `.editorconfig` - Editor formatting settings

## Architecture

### Core Model Hierarchy

All models inherit from `Base` (in `base.py`), which extends `pydantic.BaseModel` with numpy array coercion.

**Key base classes:**
- `Base` - Pydantic BaseModel with numpy handling
- `LowercaseStrEnum` - Case/hyphen/underscore-insensitive string enum
- `UniqueList` - Annotated list type that validates uniqueness

### Main Data Models

- **`Molecule`** (`molecule.py`) - Molecular structure with atoms, charge, multiplicity, and computed properties (energy, gradient, vibrational modes, thermochemistry)
- **`Atom`** (`atom.py`) - Single atom with atomic number and position

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rowansci/stjames-public](https://github.com/rowansci/stjames-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
