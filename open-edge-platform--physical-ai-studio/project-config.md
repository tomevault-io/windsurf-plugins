---
trigger: always_on
description: > **Note**: This file is synchronized with `.cursorrules`. Both Cursor and VS Code (GitHub Copilot) use these same standards.
---

# GitHub Copilot Instructions for Physical AI Studio

> **Note**: This file is synchronized with `.cursorrules`. Both Cursor and VS Code (GitHub Copilot) use these same standards.

## Table of Contents

- [GitHub Copilot Instructions for Physical AI Studio](#github-copilot-instructions-for-physical-ai-studio)
  - [Table of Contents](#table-of-contents)
  - [Project Overview](#project-overview)
  - [Coding Standards](#coding-standards)
    - [Python Environment Management](#python-environment-management)
    - [Python Code](#python-code)
    - [TypeScript/React Code](#typescriptreact-code)
    - [General Principles](#general-principles)
  - [Writing Style](#writing-style)
  - [Documentation Standards](#documentation-standards)
  - [Testing Guidelines](#testing-guidelines)
  - [File Organization](#file-organization)
  - [Git Commit Messages](#git-commit-messages)
  - [Pull Request Guidelines](#pull-request-guidelines)
  - [Performance Considerations](#performance-considerations)
  - [Security Best Practices](#security-best-practices)
  - [AI/ML Specific Guidelines](#aiml-specific-guidelines)
  - [Questions to Consider Before Coding](#questions-to-consider-before-coding)
  - [When Suggesting Code Changes](#when-suggesting-code-changes)

## Project Overview

Full-stack application with:

- **Backend**: Python FastAPI (`application/backend/`)
- **Frontend**: React/TypeScript (`application/ui/`)
- **Library**: Vision-language-action policies (`library/`)

## Coding Standards

### Python Environment Management

- **Always use `uv`** for package management and virtual environments
- Use `uv` generated virtual environments (`.venv`)
- Install with `uv pip install` or `uv sync`
- Create environments with `uv venv`
- Never use `pip` directly
- Ensure `.venv` is in `.gitignore`

### Python Code

- Follow PEP 8
- Use type hints for all functions
- Prefer `pathlib.Path` over string paths
- Use `ruff` for linting and formatting
- Address all ruff warnings

**Docstrings** - Google style format:

```python
def function_name(param1: str, param2: int) -> bool:
    """Brief description of function.

    Args:
        param1: Description of param1
        param2: Description of param2

    Returns:
        Description of return value

    Raises:
        ValueError: Description of when this is raised

    Examples:
        >>> result = function_name("test", 42)
        >>> print(result)
        True

        Multi-line example without prompt:

        from module import function_name

        result = function_name("test", 42)
        if result:
            print("Success")
    """
```

- Use `logging` instead of `print()`
- Prefer dataclasses or Pydantic models
- Use context managers for resource management

### TypeScript/React Code

- Use functional components with hooks
- Prefer named exports over default exports
- Use TypeScript strict mode with explicit types
- Follow component structure in `application/ui/src/`
- Use proper prop types and interfaces
- Implement error boundaries
- Use React Query for data fetching

### General Principles

- **DRY**: Extract common logic
- **Single Responsibility**: One clear purpose per function/class
- **Error Handling**: Handle errors with informative messages
- **Testing**: Write tests for new functionality
- **Security**: Use environment variables for secrets
- **Performance**: Consider implications, especially for ML operations

## Writing Style

Apply to code comments, documentation, commit messages, and PR descriptions:

**Be Concise**

- Remove unnecessary words
- Avoid repeating ideas
- Use 10 words instead of 20
- Prefer short sentences

**Be Direct**

- State the point immediately
- Use active voice
- Remove hedging language ("may", "might", "could potentially")

**Use Simple Language**

- Choose simple words over complex ones
- Avoid jargon unless necessary
- Break complex sentences into shorter ones

**Sound Natural**

- Write as if explaining to a colleague
- Avoid formulaic transitions ("Furthermore", "Moreover", "Additionally")
- Don't use numbered lists when a paragraph works
- Avoid: "It is important to note that", "It should be mentioned", "It is worth noting"
- Vary sentence length naturally

**Academic But Accessible**

- Use technical terms when needed, explain domain-specific ones
- Prefer clarity over impressive vocabulary

**Examples:**

❌ "It is important to note that the workshop aims to establish a comprehensive platform that serves to bring together researchers from diverse backgrounds in order to facilitate meaningful collaboration."

✅ "The workshop brings together researchers from diverse backgrounds to facilitate collaboration."

❌ "The methodology demonstrates significant improvements in terms of performance metrics."

✅ "The method improves performance."

## Documentation Standards

**Code Comments**

- Write self-documenting code
- Add comments only for the "why", not the "what"
- Update comments when code changes

**README Files**

- Each major component needs a README.md
- Include: purpose, installation, usage examples, configuration, troubleshooting

**API Documentation**

- Document endpoints with OpenAPI/Swagger
- Include request/response examples

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-edge-platform/physical-ai-studio](https://github.com/open-edge-platform/physical-ai-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
