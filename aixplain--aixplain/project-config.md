---
trigger: always_on
description: Python SDK for building, deploying, and governing AI agents on the aiXplain platform.
---

# aiXplain SDK

Python SDK for building, deploying, and governing AI agents on the aiXplain platform.

- License: Apache 2.0
- Python: >=3.9, <4
- Package config: `pyproject.toml` (PEP 621, setuptools backend)

---

## Primary Goal

- Keep the SDK stable for existing users while improving the current `development` branch.
- Default to the `v2` SDK surface for new work.
- Preserve backward compatibility for the legacy `v1` surface and legacy import paths.

---

## Source of Truth

- Package metadata and dependencies live in `pyproject.toml`.
- Formatting and docstring rules live in `ruff.toml`.
- Pre-commit behavior lives in `.pre-commit-config.yaml`.
- CI behavior lives in `.github/workflows/`.
- Public package bootstrapping and legacy import compatibility live in `aixplain/__init__.py` and `aixplain/_compat.py`.

If this file conflicts with code, tests, or CI, follow the code and tests and update this file in the same change when appropriate.

---

## Setup and Commands

```bash
# Install (development)
pip install -e .

# Install (production)
pip install aixplain

# Install with test dependencies
pip install -e ".[test]"
```

### Environment

The SDK supports either `TEAM_API_KEY` or `AIXPLAIN_API_KEY`. New code must not assume only one of those environment variables exists. `BACKEND_URL` defaults to production (`https://platform-api.aixplain.com`).

Additional environment variables for execution URLs:
- `MODELS_RUN_URL`
- `PIPELINES_RUN_URL`

In `v2`, prefer instance-scoped configuration through `Aixplain(...)` and its context rather than new global state.

### Test

```bash
# Unit tests
python -m pytest tests/unit

# Functional / integration tests
python -m pytest tests/functional

# Unit tests with coverage (same as pre-commit hook)
coverage run --source=. -m pytest tests/unit
```

### Lint and Format

Ruff is the sole linter and formatter.

```bash
ruff check .            # Lint
ruff check --fix .      # Lint with auto-fix
ruff format .           # Format
```

### Pre-commit

```bash
pre-commit install
```

Hooks run: trailing-whitespace, end-of-file-fixer, check-merge-conflict, check-added-large-files, ruff (lint + format for `aixplain/v2/`), and unit tests with coverage.

---

## Coding Conventions

- **Line length**: 120 characters.
- **Indentation**: 4 spaces.
- **Quotes**: Double quotes for strings.
- **Docstrings**: Google style (enforced by ruff `pydocstyle`). Docstring rules are **not** enforced in `tests/`.
- **Type hints**: Required on all public functions. Use `typing` (`Optional`, `Union`, `List`, `Dict`, `TypeVar`, generics).
- **Naming**: `PascalCase` for classes, `snake_case` for functions and methods, `UPPER_SNAKE_CASE` for constants.
- **Exceptions**: Use the custom hierarchy in `aixplain/exceptions/` (`AixplainBaseException` and subclasses). Never raise bare `Exception`. Preserve useful context in error messages and include status or response details when available.
- **Imports**: Use `from __future__ import annotations` or `TYPE_CHECKING` guards to break circular imports. Use conditional imports for optional dependencies. Do not add a new dependency unless it is necessary and justified by the repository's existing design.
- **Validation**: Pydantic for runtime validation. `dataclasses-json` for JSON serialization.
- **License header**: Include the Apache 2.0 license header at the top of every source file.

---

## Architecture

### Dual API Surface

The SDK exposes two API layers maintained in parallel:

| Aspect | V1 | V2 |
|---|---|---|
| Style | Factory pattern with class methods | Resource-based with dataclasses and mixins |
| Entry point | `aixplain.factories.*Factory` | `aixplain.v2.*` |
| Serialization | Manual dict handling | `dataclasses-json` (camelCase API to snake_case Python) |

### Package Layout

| Directory | Purpose |
|---|---|
| `aixplain/v2/` | Current SDK surface. Prefer this for new features and fixes unless the task is explicitly about legacy behavior. |
| `aixplain/v1/` | Legacy SDK implementation. Touch this for compatibility fixes, bug fixes, or explicitly requested v1 work. |
| `aixplain/_compat.py` | Legacy import redirector. Existing imports like `aixplain.modules` and `aixplain.factories` must continue to work. |
| `aixplain/modules/` | Domain objects (Agent, Model, Pipeline, TeamAgent, tools) |
| `aixplain/factories/` | V1 factory classes for creating and managing resources |
| `aixplain/enums/` | Enumerations (Function, Supplier, Language, Status, etc.) |
| `aixplain/exceptions/` | Custom exception hierarchy with error codes and categories |
| `aixplain/utils/` | Shared helpers (config, HTTP requests, file utilities, caching) |
| `aixplain/base/` | Base parameters |
| `aixplain/decorators/` | Decorators (e.g., API key checker) |
| `aixplain/processes/` | Data onboarding workflows |

### Key Design Patterns

- **Factory**: `AgentFactory`, `ModelFactory`, `PipelineFactory`, etc. for resource creation (V1).
- **Mixin**: `SearchResourceMixin`, `GetResourceMixin`, `RunnableResourceMixin`, `ToolableMixin` for composable behavior (V2).
- **Hook**: `before_save` / `after_save` lifecycle hooks on resources (V2).
- **Builder**: `build_run_payload()` / `build_save_payload()` methods.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aixplain/aiXplain](https://github.com/aixplain/aiXplain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
