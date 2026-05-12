---
trigger: always_on
description: A Python library for parsing and running XState-compatible JSON state machines. This is a **published PyPI library** (`xstate-statemachine`) with daily downloads. Changes must be documented in CHANGELOG.md following the Keep a Changelog format.
---

# AGENTS.md - Agent Instructions for xstate-statemachine

## Project Overview

A Python library for parsing and running XState-compatible JSON state machines. This is a **published PyPI library** (`xstate-statemachine`) with daily downloads. Changes must be documented in CHANGELOG.md following the Keep a Changelog format.

**Key Facts:**
- Python 3.8+ support
- Async-first (`Interpreter`) with sync fallback (`SyncInterpreter`)
- 100% XState JSON compatible
- Actor model support with spawn actions
- Plugin architecture for extensibility
- CLI tool (`xsm`) for code generation

## Build, Test, and Lint Commands

### Setup
```bash
# Install dependencies and set up development environment
uv pip install -e . --group dev --group lint --group test

# Install pre-commit hooks (required before first commit)
uv run pre-commit install
```

### Testing
```bash
# Run all tests
uv run pytest

# Run a single test file
uv run pytest tests/test_interpreter.py

# Run a specific test
uv run pytest tests/test_interpreter.py::TestInterpreter::test_basic_transition

# Run tests in a directory
uv run pytest tests/tests_cli/

# Run with verbose output
uv run pytest -v

# Run with coverage
uv run pytest --cov=src/xstate_statemachine
```

### Linting and Formatting
```bash
# Run all pre-commit hooks on all files
uv run pre-commit run --all-files

# Run Black formatter only
uv run black src/ tests/ --line-length=79

# Run Flake8 only
uv run flake8 src/ tests/

# Check specific file
uv run flake8 src/xstate_statemachine/interpreter.py
```

### CLI Tool
```bash
# Generate boilerplate from JSON
uv run xsm generate-template path/to/machine.json

# Using aliases
uv run xsm gt path/to/machine.json

# Hierarchical machines (parent + children)
uv run xsm gt -jp parent.json -jc child1.json -jc child2.json

# Sync mode with function style
uv run xsm gt machine.json -am no -s function

# Single file output
uv run xsm gt machine.json -fc 1
```

## Code Style Guidelines

### Imports
- **Order**: Standard library → Third-party → Project-specific
- **Format**: Use explicit imports, avoid `from module import *`
- **Group imports** with separator comments and emojis:

```python
# -------------------------------------------------------------------------
# 📦 Standard Library Imports
# -------------------------------------------------------------------------
import asyncio
import logging
from typing import Dict, List, Optional, Any

# -------------------------------------------------------------------------
# 📥 Project-Specific Imports
# -------------------------------------------------------------------------
from .models import MachineNode, StateNode
from .events import Event
```

### Formatting
- **Line length**: 79 characters (Black)
- **Formatter**: Black 25.1.0 with Python 3.13
- **Quote style**: Double quotes
- **Trailing commas**: Use in multi-line structures

### Naming Conventions
- **Python code**:
  - Functions/variables: `snake_case`
  - Classes: `PascalCase`
  - Constants: `UPPER_SNAKE_CASE`
  - Private methods: `_leading_underscore`
  - Type variables: `TContext`, `TEvent`
- **JSON (XState config)**:
  - Actions/guards/services: `camelCase` (auto-converted from Python `snake_case`)
  - Events: `SCREAMING_SNAKE_CASE`
  - State IDs: `camelCase`

### Type Hints
- Use comprehensive type hints for all function signatures
- Use generics where appropriate: `MachineNode[TContext, TEvent]`
- Import from `typing`: `Dict`, `List`, `Optional`, `Any`, `Callable`, `Union`, etc.
- Use `-> None` for functions that don't return

### Error Handling
- **Guards**: Must return `bool`. If guard raises exception, it's treated as `False`
- **Actions**: Log errors, skip remaining actions in list, continue processing
- **Services**: Raise exceptions to trigger `onError` transition
- **Exceptions**: Use custom exception hierarchy (all inherit from `XStateMachineError`)

```python
# Guard example - pure function, returns bool
def can_retry(ctx: Dict, event: Event) -> bool:
    return ctx.get("attempts", 0) < 3

# Action example - can mutate context, no return
def increment_retry(i, ctx: Dict, e: Event, a: ActionDefinition) -> None:
    ctx["attempts"] = ctx.get("attempts", 0) + 1
    logger.info("Retry count: %d", ctx["attempts"])

# Service example - can raise exceptions
def fetch_data(i, ctx: Dict, e: Event) -> Dict:
    result = api_call()  # May raise
    return result
```

### Documentation
- **File headers**: Include module docstrings with emoji markers and description
- **Function docstrings**: Google-style with Args/Returns sections
- **Class docstrings**: Describe purpose, attributes, and usage
- **Inline comments**: Use emoji prefixes for quick visual grep:
  - `✅` Success/completion
  - `🚀` Initialization
  - `🔥` Errors/warnings
  - `💡` Tips/hints
  - `⚠️` Caution
  - `📝` Notes/explanations
  - `🏛️` Architecture decisions

```python
def process_event(self, event: Event) -> None:
    """Process an event through the state machine.

    Args:
        event: The event to process, containing type and payload.

    Returns:
        None

    Raises:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [basiltt/xstate-statemachine](https://github.com/basiltt/xstate-statemachine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
