---
trigger: always_on
description: Guidelines for agentic coding assistants working on this codebase.
---

# AGENTS.md — Cognition Coding Agent

Guidelines for agentic coding assistants working on this codebase.

## Project Overview

Cognition is an OpenCode-style coding agent with:
- **Server**: FastAPI WebSocket API with Deep Agents runtime (`server/`)
- **Client**: CLI/TUI for interactive sessions (`client/`)
- **Execution**: Container-per-session with optional network isolation
- **Scope**: Python repos only, pytest-based testing

## Build / Test / Lint Commands

This project uses `uv` for dependency management and task execution.

```bash
# Install dependencies
uv sync

# Run server (development)
# Runs the FastAPI server with hot reload
uv run uvicorn server.app.main:app --reload --port 8000

# Run client (development)
# Starts the CLI/TUI client
uv run python -m client.cli.main

# Run all tests
uv run pytest

# Run single test file
uv run pytest tests/unit/test_settings.py -v

# Run single test case
uv run pytest tests/unit/test_settings.py::TestSettingsDefaults::test_default_server_settings -v

# Type checking (Strict)
uv run mypy .

# Linting & Formatting (Ruff)
uv run ruff check .
uv run ruff format .
```

## Code Style Guidelines

### Python Standards
- **Python 3.11+** required.
- **Type Hints**: strict `mypy` compliance. Use `from __future__ import annotations`.
- **Docstrings**: Google style for all public functions/classes.
- **Line Length**: Follow `ruff` config (default 88/100).
- **Imports**: Grouped as stdlib, third-party, local. Use absolute imports.

### Naming Conventions
- `snake_case`: functions, variables, modules.
- `PascalCase`: classes, types.
- `UPPER_CASE`: constants.
- `_prefix`: private methods/attributes.
- `async` functions: often prefixed with verbs like `get_`, `fetch_`, `handle_`.

### Async Patterns
- **Async/Await**: Use for all I/O operations (DB, Network, File).
- **Concurrency**: Use `asyncio.gather()` for parallel independent tasks.
- **Subprocesses**: Use `asyncio.create_subprocess_exec`.

### Error Handling
- **Exceptions**: Use custom hierarchy in `server/app/exceptions.py`.
- **Pattern**: Catch external errors -> Raise domain-specific `CognitionError`.
- **Logging**: Log errors before raising if context is needed, otherwise let middleware handle it.

### Data Models
- **Pydantic V2**: Use for all data structures and validation.
- **Settings**: Use `pydantic-settings` (e.g., `server/app/settings.py`).
- **Secrets**: Use `SecretStr` for sensitive data.

## Project Structure

```
cognition/
├── server/
│   └── app/
│       ├── agent/       # Deep Agents runtime & tools
│       ├── api/         # FastAPI routes
│       ├── llm/         # LLM service integration
│       └── persistence/ # Database/Storage
├── client/
│   └── cli/             # CLI/TUI entry points
├── tests/
│   ├── e2e/             # End-to-end full workflow tests
│   └── unit/            # Isolated unit tests
├── pyproject.toml       # Project dependencies & tool config
└── uv.lock              # Dependency lock file
```

## Key Workflows

### Extending Cognition
Cognition is designed to be highly pluggable using native `deepagents` extension points. Deep Agents is the higher-level abstraction that Cognition is built on — it wraps LangGraph and provides its own primitives (tools, middleware, skills, subagents, memory, sandbox backends). We should prefer Deep Agents' API surface over reaching down to raw LangGraph.

#### Custom Tools
1. Define your tool as a plain Python callable or LangChain `BaseTool`.
2. Register it in `.cognition/config.yaml` under `agent.tools` or pass it to `create_cognition_agent(tools=[...])`.

#### Agent Middleware
1. Implement `deepagents.middleware.AgentMiddleware` for lifecycle hooks (observability, status streaming, etc.).
2. Add to `create_cognition_agent(middleware=[...])`.

**Available upstream middleware** (declarative in `.cognition/agent.yaml`):
- `tool_retry` - Exponential backoff retry on tool failure
- `tool_call_limit` - Per-tool and global call limits
- `human_in_the_loop` - Approve/edit/reject tool calls before execution
- `pii` - Detect and redact PII (email, credit card, IP, etc.)

#### Skills & Memory
1. Add `SKILL.md` files to `.cognition/skills/` for progressive disclosure capabilities.
2. Update `AGENTS.md` in the workspace to provide the agent with project-specific conventions.

#### Subagents
1. Define specialized subagents in `.cognition/config.yaml` to handle complex domain-specific tasks in isolated contexts.

### Releases
- Follow `docs/guides/release-checklist.md` for all release preparation, validation, tagging, and post-release verification.
- Do not cut or replace a release tag until the exact release commit has passed code-quality and container-build validation.
- Before tagging, run the pre-release image workflow for the exact release branch commit so app and sandbox multi-arch pushes are proven against GHCR.

### Testing & Scenarios
- **Unit Tests**: Fast, mocked dependencies. No containers.
- **E2E Tests**: Use `tests/e2e/`. May require running server.
- **Mocking**: Use `unittest.mock` or `pytest-mock` for external services (LLMs).
- **Scenarios**: Scenarios are business logic use case against the APIs. They should be tested in E2E tests with realistic inputs against the docker-compose environment.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CognicellAI/Cognition](https://github.com/CognicellAI/Cognition) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
