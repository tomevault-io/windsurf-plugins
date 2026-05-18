---
trigger: always_on
description: This document provides comprehensive guidelines for AI coding agents contributing to the mlx-openai-server project. Following these standards ensures consistency, maintainability, and quality across the codebase.
---

# AI Agent Development Guidelines

This document provides comprehensive guidelines for AI coding agents contributing to the mlx-openai-server project. Following these standards ensures consistency, maintainability, and quality across the codebase.

---

## Table of Contents

1. [Environment Setup](#environment-setup)
2. [Code Quality & Style](#code-quality--style)
3. [Python Best Practices](#python-best-practices)
4. [Error Handling & Logging](#error-handling--logging)
5. [Version Control & Collaboration](#version-control--collaboration)
6. [Testing Requirements](#testing-requirements)
7. [Transparency & Communication](#transparency--communication)

---

## Environment Setup

### Virtual Environment

- **Repository-local environment**: Agents may create and use a virtual environment located at `./.venv` in the repository root.
- **Python interpreter**: All Python commands, scripts, and tests **must** use the interpreter at `./.venv/bin/python`. Never use the system Python or global environment.
- **Activation**: When running terminal commands, explicitly invoke `./.venv/bin/python` or `./.venv/bin/<tool>` rather than relying on shell activation.

### Package Management

- **Manifest-based installation**: Agents are permitted to install packages declared in repository manifests (e.g., `pyproject.toml`, `requirements.txt`) into the local `.venv` without requiring additional explicit approval for each installation.
- **Global environment prohibition**: Under no circumstances should packages be installed into the system or global Python environment. All dependencies must remain isolated within `./.venv`.
- **Dependency updates**: When adding new dependencies, update the appropriate manifest file (`pyproject.toml` preferred) and document the reason in commit messages or PR descriptions.

---

## Code Quality & Style

### Linting & Formatting Tools

This project uses the following tools to maintain consistent code quality:

- **ruff**: Automatic code formatter with options configured in `pyproject.toml`

### Pre-commit Hooks

This project uses [pre-commit](https://pre-commit.com/) to enforce code quality. All hooks **must** pass before submitting a pull request.

**Install hooks (one-time setup):**

```bash
pre-commit install
```

**Run all hooks manually:**

```bash
pre-commit run --all-files
```

Pre-commit runs automatically on `git commit` once installed. The configured hooks include: trailing whitespace, end-of-file fixer, YAML/TOML validation, shellcheck, codespell, and ruff (linting + formatting).

### Formatting Workflow

You can also run ruff directly on specific files:

```bash
./.venv/bin/ruff check --fix <file_or_directory>
./.venv/bin/ruff format <file_or_directory>
```

---

## Python Best Practices

### Type Annotations

- **Mandatory typing**: Add type annotations to all function signatures, method signatures, and class attributes.
- **Return types**: Always specify return types, including `None` when applicable.
- **Minimize `Any`**: Do not just use `Any` for typing to make the error go away. Use appropriate type annotations and only use `Any` when applicable (ex. if there are > 3 different return types possible).
- **Forward references**: Use `from __future__ import annotations` to defer evaluation of type annotations, allowing forward references without string literals.
- **Python 3.11+ type hints**: Use built-in generic types instead of typing module equivalents (e.g., `dict[str, Any]` instead of `Dict[str, Any]`, `list[str]` instead of `List[str]`).

**Example:**

```python
from __future__ import annotations  # Modern approach: defers evaluation, no string literals needed
from typing import Any

def process_request(
    request_id: str, 
    data: dict[str, Any], 
    timeout: float | None = None
) -> list[Response]:
    """Process a request and return results."""
    ...

class Response:
    def __init__(self, status: str) -> None:
        self.status = status

# Example of self-referential type (forward reference)
class Node:
    def __init__(self, value: str, children: list[Node] | None = None) -> None:
        self.value = value
        self.children = children or []

# Alternative with string literals (for older Python or when not using __future__ import):
# class Node:
#     def __init__(self, value: str, children: list["Node"] | None = None) -> None:
#         self.value = value
#         self.children = children or []
```

### Documentation

- **Docstrings required**: All modules, classes, and methods must have descriptive docstrings.
- **Docstring format**: Use triple double-quotes. Method docstrings must follow NumPy style, with sections for Parameters, Returns, and Raises where applicable. The first line should be a concise summary; additional lines may include extended descriptions.
- **Update existing docstrings**: If modifying a function's behavior, update its docstring to reflect changes.

**Example:**

```python
def calculate_metrics(data: list[float], threshold: float) -> dict[str, float]:
    """Calculate statistical metrics for the provided data.
    
    Parameters
    ----------
    data : list[float]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cubist38/mlx-openai-server](https://github.com/cubist38/mlx-openai-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
