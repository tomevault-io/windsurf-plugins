---
trigger: always_on
description: Lightspeed Core Stack (LCS) is an AI-powered assistant built on FastAPI that provides answers using LLM services, agents, and RAG databases. It integrates with Llama Stack for AI operations.
---

# Lightspeed Core Stack Development Guide

## Project Overview
Lightspeed Core Stack (LCS) is an AI-powered assistant built on FastAPI that provides answers using LLM services, agents, and RAG databases. It integrates with Llama Stack for AI operations.

## Development Environment
- **Python**: Check `pyproject.toml` for supported Python versions
- **Package Manager**: uv (use `uv run` for all commands)
- **Required Commands**:
  - `uv run make format` - Format code (black + ruff)
  - `uv run make verify` - Run all linters (black, pylint, pyright, ruff, docstyle, check-types)

## Code Architecture & Patterns

### Project Structure
```
src/
├── app/                  # FastAPI application
│   ├── endpoints/        # REST API endpoints
│   └── main.py           # Application entry point
├── a2a_storage/          # A2A protocol persistent storage
│   ├── context_store.py  # Abstract base class for context stores
│   ├── in_memory_context_store.py  # In-memory implementation
│   ├── sqlite_context_store.py     # SQLite implementation
│   ├── postgres_context_store.py   # PostgreSQL implementation
│   └── storage_factory.py          # Factory for creating stores
├── authentication/       # Authentication modules (k8s, jwk, noop, rh-identity)
├── authorization/        # Authorization middleware & resolvers
├── cache/                # Conversation cache implementations
├── quota/                # Quota limiter and token usage tracking
├── metrics/              # Prometheus metrics
├── runners/              # Agent runners
├── models/               # Pydantic models
│   ├── api/
│   │   ├── requests/     # REST API request models
│   │   └── responses/    # HTTP response models
│   ├── common/           # Shared cross-layer models
│   └── database/         # Database models
├── utils/                # Utility functions
├── client.py             # Llama Stack client wrapper (Singleton)
├── configuration.py      # Config management (Singleton)
└── constants.py          # Shared constants
```

### Coding Standards

#### Imports & Dependencies
- Use absolute imports for internal modules: `from authentication import get_auth_dependency`
- FastAPI dependencies: `from fastapi import APIRouter, HTTPException, Request, status, Depends`
- Llama Stack imports: `from llama_stack_client import AsyncLlamaStackClient`
- **ALWAYS** check `pyproject.toml` for existing dependencies before adding new ones
- **ALWAYS** verify current library versions in `pyproject.toml` rather than assuming versions
- Check `constants.py` for shared constants before defining new ones

#### Module Standards
- All modules start with descriptive docstrings explaining purpose
- Use `logger = get_logger(__name__)` from `log.py` for module logging
- Package `__init__.py` files contain brief package descriptions
- Central `constants.py` for shared constants with descriptive comments
- Type aliases defined at module level for clarity
- Use Final[type] as type hint for all constants

#### Configuration
- All config uses Pydantic models extending `ConfigurationBase`
- Base class sets `extra="forbid"` to reject unknown fields
- Use `@field_validator` and `@model_validator` for custom validation
- Type hints: `Optional[FilePath]`, `PositiveInt`, `SecretStr`

#### Function Standards
- **Documentation**: All functions require docstrings with brief descriptions
- **Type Annotations**: Complete type annotations for parameters and return types
  - Use `typing_extensions.Self` for model validators
  - Union types: `str | int` (modern syntax)
  - Optional: `Optional[Type]`
- **Naming**: Use snake_case with descriptive, action-oriented names (get_, validate_, check_)
- **Return Values**: **CRITICAL** - Avoid in-place parameter modification anti-patterns:
  ```python
  # ❌ BAD: Modifying parameter in-place
  def process_data(input_data: Any, result_dict: dict) -> None:
      result_dict[key] = value  # Anti-pattern

  # ✅ GOOD: Return new data structure
  def process_data(input_data: Any) -> dict:
      result_dict = {}
      result_dict[key] = value
      return result_dict
  ```
- **Async Functions**: Use `async def` for I/O operations and external API calls
- **Error Handling**:
  - Use FastAPI `HTTPException` with appropriate status codes for API endpoints
  - Handle `APIConnectionError` from Llama Stack

#### Logging Standards
- Use `from log import get_logger` and module logger pattern: `logger = get_logger(__name__)`
- Standard log levels with clear purposes:
  - `logger.debug()` - Detailed diagnostic information
  - `logger.info()` - General information about program execution
  - `logger.warning()` - Something unexpected happened or potential problems
  - `logger.error()` - Serious problems that prevented function execution

#### Class Standards
- **Documentation**: All classes require descriptive docstrings explaining purpose
- **Naming**: Use PascalCase with descriptive names and standard suffixes:
  - `Configuration` for config classes
  - `Error`/`Exception` for custom exceptions
  - `Resolver` for strategy pattern implementations
  - `Interface` for abstract base classes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lightspeed-core/lightspeed-stack](https://github.com/lightspeed-core/lightspeed-stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
