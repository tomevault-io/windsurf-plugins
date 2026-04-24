---
trigger: always_on
description: This guide covers best practices for contributing to the core Recursive Language Models `rlm` library and developing new environments (in `rlm/environments/`) and LM clients (in `rlm/clients/`).
---

# AGENTS.md

This guide covers best practices for contributing to the core Recursive Language Models `rlm` library and developing new environments (in `rlm/environments/`) and LM clients (in `rlm/clients/`).

## Setup

We use `uv` for developing `rlm`.
```bash
# Install uv (first time)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Setup blank project if needed
uv init && uv venv --python 3.12
source .venv/bin/activate

# Install in editable mode
uv pip install -e .

# For Modal sandbox support
uv pip install -e ".[modal]"
```

## General Guidelines

### Code Style & Typing
- **Formatting**: Strict `ruff` enforcement. All PRs must pass `ruff check --fix .`
- **Typing**: Explicit types preferred
  - **OK**: `cast(...)`, `assert ...` for type narrowing
  - **SOMETIMES OK**: Untyped args for simple cases (e.g., prompt handlers)
  - **NOT OK**: `# type: ignore` without strong justification

### Naming Conventions
- **Methods**: snake_case
- **Classes**: PascalCase (e.g., `LocalREPL`, `PortkeyClient`)
- **Variables**: snake_case
- **Constants**: UPPER_CASE (e.g., `_SAFE_BUILTINS`, `RLM_SYSTEM_PROMPT`)

Do NOT use `_` prefix for private methods unless explicitly requested.

### Error Handling Philosophy
- **Fail fast, fail loud** - No defensive programming or silent fallbacks
- **Minimize branching** - Prefer single code paths; every `if`/`try` needs justification
- **Example**: Missing API key → immediate `ValueError`, not graceful fallback

## Core Repository Development

For PRs to `rlm` core:
```bash
git clone https://github.com/alexzhang13/rlm.git
cd rlm

# Standard development:
uv sync

# Install dev + test dependencies:
uv sync --group dev --group test

# Install pre-commit hooks:
uv run pre-commit install
```

### Dependencies
- Avoid new core dependencies
- Use optional extras for non-essential features (e.g., `modal` extra)
- Exception: tiny deps that simplify widely-used code

### Testing
- `uv run pytest` with discovery under `tests/`
- Write simple, deterministic unit tests
- Update tests when changing functionality
- For isolated environments, mock external services

### Documentation
- Keep concise and actionable
- Update README when behavior changes
- Avoid content duplication

### Scope
- Small, focused diffs
- One change per PR
- Backward compatibility is only desirable if it can be done without introducing excessive maintenance burden
- Delete dead code (don't guard it)

### Checklist

Before a PR:

```bash
# Run style + lint checks:
uv run ruff check --fix .
uv run ruff format .
uv run pre-commit run --all-files

# Run tests:
uv run pytest
```

Ensure docs and tests are updated if necessary, and dead code is deleted. Strive for minimal, surgical diffs.

## Developing LM Clients

LM client implementations live in `rlm/clients/`. All clients must inherit from `BaseLM`.

### Client Pattern

| Base Class | When to Use | Key Methods |
|------------|-------------|-------------|
| `BaseLM` | All LM integrations | `completion`, `acompletion`, `get_usage_summary`, `get_last_usage` |

### Requirements
- Inherit from `BaseLM` in `rlm/clients/base_lm.py`
- Implement all abstract methods: `completion`, `acompletion`, `get_usage_summary`, `get_last_usage`
- Track per-model usage (calls, input/output tokens)
- Handle both string and message list prompts
- Register client in `rlm/clients/__init__.py`

### Example Structure
```python
from rlm.clients.base_lm import BaseLM
from rlm.core.types import ModelUsageSummary, UsageSummary

class MyClient(BaseLM):
    def __init__(self, api_key: str, model_name: str, **kwargs):
        super().__init__(model_name=model_name, **kwargs)
        # Initialize your client
        
    def completion(self, prompt: str | list[dict[str, Any]], model: str | None = None) -> str:
        # Handle both str and message list formats
        # Track usage with _track_cost()
        # Return response string
        
    def get_usage_summary(self) -> UsageSummary:
        # Return aggregated usage across all calls
```

### Configuration Guidelines
- **Environment variables**: ONLY for API keys (document in README)
- **Hardcode**: Default base URLs, reasonable defaults
- **Arguments**: Essential customization via `__init__()`

## Developing Environments

Environment implementations live in `rlm/environments/`. Choose the appropriate base class.

### Environment Pattern

| Pattern | Base Class | When to Use | Key Methods |
|---------|------------|-------------|-------------|
| **Non-isolated** | `NonIsolatedEnv` | Local execution, same machine | `setup`, `load_context`, `execute_code` |
| **Isolated** | `IsolatedEnv` | Cloud sandboxes (Modal, Prime) | `setup`, `load_context`, `execute_code` |

### Requirements
- Inherit from `NonIsolatedEnv` or `IsolatedEnv` in `rlm/environments/base_env.py`
- Implement all abstract methods: `setup`, `load_context`, `execute_code`
- Return `REPLResult` from `execute_code`
- Handle `lm_handler_address` for sub-LM calls via `llm_query()`
- Implement `cleanup()` for resource management
- Register environment in `rlm/environments/__init__.py`

### Key Implementation Details

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mitkox/rlmgw](https://github.com/mitkox/rlmgw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
