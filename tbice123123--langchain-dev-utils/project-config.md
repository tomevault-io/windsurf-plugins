---
trigger: always_on
description: This file provides guidelines for AI agents working in this repository.
---

# Agentic Coding Guidelines for langchain-dev-utils

This file provides guidelines for AI agents working in this repository.

## Project Overview

A Python utility library for LangChain and LangGraph development. Uses hatchling build system, uv for package management, ruff for linting, and pytest for testing.

**Current Version:** 1.4.6

---

## Build/Lint/Test Commands

### Package Management (uv)
```bash
# Install dependencies
uv sync

# Install with optional dependencies
uv sync --extra standard

# Install dev dependencies
uv sync --group dev

# Install test dependencies
uv sync --group tests

# Install docs dependencies
uv sync --group docs
```

### Linting & Formatting (ruff)
```bash
# Check linting
uv run ruff check .

# Check specific file
uv run ruff check src/langchain_dev_utils/path/to/file.py

# Fix auto-fixable issues
uv run ruff check --fix .

# Format code
uv run ruff format .

# Format specific file
uv run ruff format src/langchain_dev_utils/path/to/file.py
```

### Testing (pytest)
```bash
# Run all tests
uv run pytest

# Run with verbose output
uv run pytest -v

# Run specific test file
uv run pytest tests/test_agent.py

# Run specific test function
uv run pytest tests/test_agent.py::test_prebuilt_agent

# Run specific test class
uv run pytest tests/test_chat_models.py::TestImageProcessing

# Run with asyncio support (automatically configured)
uv run pytest -s

# Run tests matching pattern
uv run pytest -k "test_load"
```

### Build
```bash
# Build package
uv build

# Build wheel only
uv build --wheel
```

### Documentation
```bash
# Serve docs locally
mkdocs serve

# Build docs
mkdocs build

# Deploy docs
mkdocs gh-deploy
```

---

## Code Style Guidelines

### Imports
- Use absolute imports for external packages
- Use relative imports within the package (e.g., `from ..chat_models import ...`)
- Group imports: stdlib → third-party → local
- Ruff handles import sorting automatically

### Type Hints
- Use type hints for all function parameters and return types
- Use `Optional[Type]` or `Type | None` for nullable types (both acceptable)
- Use `Any` sparingly and only when necessary
- Use `Sequence`, `Mapping` for generic collections
- Use generics with TypeVars where appropriate

### Naming Conventions
- `snake_case` for functions, methods, variables
- `PascalCase` for classes
- `UPPER_CASE` for constants
- `_leading_underscore` for private/internal functions
- Leading double underscore for name mangling when needed

### Docstrings
- Use Google-style docstrings
- Include Args, Returns, Raises sections for public functions
- Include Examples section for complex functions
- Keep docstrings under 100 characters per line when possible

### Code Structure
- Maximum line length: 88 characters (ruff enforces, E501 ignored)
- Use trailing commas in multi-line structures
- Two blank lines between top-level functions/classes
- One blank line between methods

### Error Handling
- Use specific exception types, not bare `except:`
- Provide descriptive error messages with f-strings
- Use `raise ValueError(msg)` pattern with descriptive messages
- Avoid bare `raise` statements

### Async Code
- Use `pytest.mark.asyncio` for async test functions
- Use `async`/`await` consistently
- Prefer `asyncio` primitives from standard library

### Ruff Configuration
- Enabled rules: E, F, I, PGH003, T201
- Import sorting (I) is enforced
- No print statements in production code (T201)

---

## Testing Guidelines

- Tests live in `tests/` directory
- Test files named `test_*.py`
- Test functions named `test_*`
- Use pytest fixtures for setup/teardown
- Use `pytest.mark.asyncio` for async tests
- Mock external API calls in unit tests
- Integration tests use actual APIs (marked implicitly by test names)
- Tests use `langchain-tests` for standard integration test suites

---

## Project Structure

```
langchain-dev-utils/
├── src/langchain_dev_utils/      # Source code
│   ├── agents/                   # Agent utilities
│   │   ├── factory.py            # Agent creation functions
│   │   ├── wrap.py               # Agent wrapping utilities
│   │   └── middleware/           # Agent middleware components
│   │       ├── format_prompt.py      # Format prompt middleware
│   │       ├── handoffs.py           # Multi-agent handoff middleware
│   │       ├── model_fallback.py     # Model fallback middleware
│   │       ├── model_router.py       # Model routing middleware
│   │       ├── plan.py               # Planning middleware
│   │       ├── summarization.py      # Message summarization middleware
│   │       ├── tool_call_repair.py   # Tool call repair middleware
│   │       ├── tool_emulator.py      # LLM tool emulator middleware
│   │       └── tool_selection.py     # LLM tool selector middleware
│   ├── chat_models/              # Chat model utilities
│   │   ├── base.py               # Base chat model classes
│   │   ├── types.py              # Type definitions
│   │   └── adapters/             # Model provider adapters
│   ├── embeddings/               # Embedding utilities
│   │   ├── base.py               # Base embedding classes
│   │   └── adapters/             # Embedding provider adapters
│   ├── graph/                    # Graph utilities

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TBice123123/langchain-dev-utils](https://github.com/TBice123123/langchain-dev-utils) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
