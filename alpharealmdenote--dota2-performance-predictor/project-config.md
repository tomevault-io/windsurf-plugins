---
trigger: always_on
description: This file provides guidance to AI Agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI Agents when working with code in this repository.

## Taskfile (Recommended)

This project uses [Task](https://taskfile.dev/) as a unified command runner. All build, dev, test, lint, and docker commands can be run from the repo root via `task <command>`. Run `task --list` to see all available commands.

Task `desc:` fields should describe **what** the task does, not **how** it does it. Keep them generic and stable: don't reference implementation details like aliases, internal helpers, mode flags, or which other task delegates to which. The description is for users picking a command from `task --list`, not a changelog of refactors.

### Quick Reference
- `task install` — install all dependencies
- `task dev` — start backend + frontend concurrently
- `task dev:all` — start backend + frontend + engine concurrently
- `task build` — build all components
- `task test` — run all tests (backend + frontend + engine)
- `task lint` — run all linters
- `task format` — auto-fix formatting across all components
- `task check` — full quality gate (lint + typecheck + test)
- `task clean` — clean all build artifacts
- `task docker:build` — build standard Docker image
- `task docker:up` — start Docker compose stack

## Common Development Commands

### Build and Test
- **Build project**: `task build`
- **Run backend locally**: `task backend:dev`
- **Run all tests**: `task test` (or individually: `task backend:test`, `task frontend:test`, `task engine:test`)
- **Docker integration tests**: `./test.sh` (builds all Docker variants and runs comprehensive tests)
- **Code formatting**: `task format` (or `task backend:format` for Java only)
- **Full quality gate**: `task check` (runs lint + typecheck + test across all components)

After modifying any files in the project, you must run the relevant `task check` command that covers that area of the code. For example, when editing frontend files run `task frontend:check`; for Python engine files run `task engine:check`; for Java backend files run `task backend:check`.

### Docker Development
- **Build standard**: `task docker:build` (or `docker build -t stirling-pdf -f docker/embedded/Dockerfile .`)
- **Build fat version**: `task docker:build:fat`
- **Build ultra-lite**: `task docker:build:ultra-lite`
- **Start compose stack**: `task docker:up` (or `task docker:up:fat`, `task docker:up:ultra-lite`)
- **Stop compose stack**: `task docker:down`
- **View logs**: `task docker:logs`
- **Example compose files**: Located in `exampleYmlFiles/` directory

### Security Mode Development
Set `DOCKER_ENABLE_SECURITY=true` environment variable to enable security features during development. This is required for testing the full version locally.

### Python Development (AI Engine)

The engine is a Python reasoning service for Stirling: it plans and interprets work, but it does not own durable state, and it does not execute Stirling PDF operations directly. Keep the service narrow: typed contracts in, typed contracts out, with AI only where it adds reasoning value. The frontend calls the Python engine via Java as a proxy.

#### Python Commands
All engine commands run from the repo root using Task:
- `task engine:check` — run all checks (typecheck + lint + format-check + test)
- `task engine:fix` — auto-fix lint + formatting
- `task engine:install` — install Python dependencies via uv
- `task engine:dev` — start FastAPI with hot reload (localhost:5001)
- `task engine:test` — run pytest
- `task engine:lint` — run ruff linting
- `task engine:typecheck` — run pyright
- `task engine:format` — format code with ruff
- `task engine:tool-models` — generate `tool_models.py` from the Java OpenAPI spec

The project structure is defined in `engine/pyproject.toml`. Any new dependencies should be listed there, followed by running `task engine:install`.

#### Python Code Style
- Keep `task engine:check` passing.
- Use modern Python when it improves clarity.
- Prefer explicit names to cleverness.
- Avoid nested functions and nested classes unless the language construct requires them.
- Prefer composition to inheritance when combining concepts.
- Avoid speculative abstractions. Add a layer only when it removes real duplication or clarifies lifecycle.
- Add comments sparingly and only when they explain non-obvious intent.

#### Python Typing and Models
- Deserialize into Pydantic models as early as possible.
- Serialize from Pydantic models as late as possible.
- Do not pass raw `dict[str, Any]` or `dict[str, object]` across important boundaries when a typed model can exist instead.
- Avoid `Any` wherever possible.
- Avoid `cast()` wherever possible (reconsider the structure first).
- All shared models should subclass `stirling.models.ApiModel` so the service behaves consistently.
- Do not use string literals for any type annotations, including `cast()`.

#### Python Configuration
- Keep application-owned configuration in `stirling.config`.
- Only add `STIRLING_*` environment variables that the engine itself truly owns.
- Do not mirror third-party provider environment variables unless the engine is actually interpreting them.
- Let `pydantic-ai` own provider authentication configuration when possible.

#### Python Architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alpharealmdenote/Dota2-Performance-Predictor](https://github.com/alpharealmdenote/Dota2-Performance-Predictor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-08 -->
