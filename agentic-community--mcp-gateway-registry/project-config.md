---
trigger: always_on
description: This document contains coding standards and best practices that must be followed for all code development. These rules prioritize maintainability, simplicity, and modern Python development practices.
---

# Claude Coding Rules

## Overview
This document contains coding standards and best practices that must be followed for all code development. These rules prioritize maintainability, simplicity, and modern Python development practices.

## Core Principles
- Write code with minimal complexity for maximum maintainability and clarity
- Choose simple, readable solutions over clever or complex implementations
- Prioritize code that any team member can confidently understand, modify, and debug

## Pull Request Evaluation

When evaluating pull requests for merge, adopt the **Merge Specialist** persona defined in [TEAM.md](TEAM.md). This persona provides comprehensive guidelines for:

- Running and verifying tests
- Assessing code quality against these standards
- Reviewing architecture and design decisions
- Checking for breaking changes
- Evaluating performance impact
- Ensuring documentation is complete

**IMPORTANT**: Before approving any PR for merge, the Merge Specialist must verify that all tests pass and no existing functionality is broken. A PR with failing tests should NEVER be approved for merge.

## Technology Stack

### Package Management
- Always use `uv` and `pyproject.toml` for package management
- Never use `pip` directly

### Modern Python Libraries
- **Data Processing**: Use `polars` instead of `pandas`
- **Web APIs**: Use `fastapi` instead of `flask`
- **Code Formatting/Linting**: Use `ruff` for both linting and formatting
- **Type Checking**: Use `mypy` - type checks have become actually useful and should be part of CI/CD
- **Performance**: Leverage modern CPython improvements - CPython is now much faster

## Code Style Guidelines

### Function Structure
- All internal/private functions must start with an underscore (`_`)
- Private functions should be placed at the top of the file, followed by public functions
- Functions should be modular, containing no more than 30-50 lines
- Use two blank lines between function definitions
- One function parameter per line for better readability

### Type Annotations
- Use clear type annotations for all function parameters
- One function parameter per line for better readability
- Use modern Python 3.10+ type hint syntax (PEP 604/585)
- Example:
  ```python
  def process_data(
      input_file: str,
      output_format: str,
      validate: bool = True
  ) -> dict[str, Any]:
      pass
  ```

### Modern Type Hint Standards (Python 3.10+)

**IMPORTANT**: This codebase uses modern Python 3.10+ type hint syntax (PEP 604 and PEP 585). Always use built-in types instead of importing from `typing` module.

#### PEP 604: Union Types with `|`
Use `X | None` instead of `Optional[X]`:

```python
# Good - Modern syntax (Python 3.10+)
def process_data(
    sample_size: int | None = None,
    language: str | None = None
) -> list[dict[str, Any]]:
    pass

# Avoid - Legacy syntax
from typing import Optional, List, Dict, Any

def process_data(
    sample_size: Optional[int] = None,
    language: Optional[str] = None
) -> List[Dict[str, Any]]:
    pass
```

#### PEP 585: Built-in Generic Types
Use `list`, `dict`, `tuple`, `set` directly instead of importing from `typing`:

```python
# Good - Built-in generic types
def process_items(
    data: list[dict[str, Any]],
    filters: set[str],
    metadata: tuple[str, int]
) -> dict[str, list[Any]]:
    pass

# Avoid - typing module imports
from typing import List, Dict, Set, Tuple, Any

def process_items(
    data: List[Dict[str, Any]],
    filters: Set[str],
    metadata: Tuple[str, int]
) -> Dict[str, List[Any]]:
    pass
```

#### Type Hint Migration Examples

**Example 1: Optional Parameters**
```python
# Old style
from typing import Optional

def get_user(user_id: int, token: Optional[str] = None) -> Optional[dict]:
    pass

# New style - no imports needed
def get_user(user_id: int, token: str | None = None) -> dict | None:
    pass
```

**Example 2: Complex Types**
```python
# Old style
from typing import List, Dict, Optional, Tuple

def process_samples(
    sample_size: Optional[int] = None,
    language: Optional[str] = None
) -> List[dict]:
    """Process dataset samples.

    Args:
        sample_size: Number of samples. None uses default, 0 means all.
        language: Language filter. None means all languages.
    """
    if sample_size == 0:
        return process_all()
    elif sample_size is None:
        sample_size = DEFAULT_SAMPLE_SIZE

    return process_with_size(sample_size)

# New style - cleaner and more Pythonic
def process_samples(
    sample_size: int | None = None,
    language: str | None = None
) -> list[dict[str, Any]]:
    """Process dataset samples.

    Args:
        sample_size: Number of samples. None uses default, 0 means all.
        language: Language filter. None means all languages.
    """
    if sample_size == 0:
        return process_all()
    elif sample_size is None:
        sample_size = DEFAULT_SAMPLE_SIZE

    return process_with_size(sample_size)
```

**Example 3: Nested Generic Types**
```python
# Old style
from typing import Dict, List, Tuple, Optional

def get_user_data(
    user_id: int

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentic-community/mcp-gateway-registry](https://github.com/agentic-community/mcp-gateway-registry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
