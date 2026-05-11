---
trigger: always_on
description: This is the LlamaIndex Workflows library - an event-driven, async-first framework for orchestrating complex AI applications and multi-step processes.
---

# LlamaIndex Workflows - Claude Development Guide

## Project Overview
This is the LlamaIndex Workflows library - an event-driven, async-first framework for orchestrating complex AI applications and multi-step processes.

## Key Technologies
- Python 3.9+
- AsyncIO (async/await)
- Pydantic for data models
- Starlette for web server
- Uvicorn for ASGI serving

## Setup
- Install uv: `curl -LsSf https://astral.sh/uv/install.sh | sh`
- Install deps (dev): `uv sync --all-packages --all-extras`

## Development Commands

### Testing

Use the `dev` CLI to run tests:

```bash
# Run all package tests
uv run dev

# Filter by substring match
uv run dev -p workflows
uv run dev -p server -p client

# Pass pytest args after --
uv run dev -- -k test_name
```

For more advanced scenarios, you can always `cd packages/some-package` and use pytest directly. The dev tool just provides additional package level test parallelism, and more curated cross package test output to avoid context bloat.

Several packages have Docker integration tests (requires Docker running) marked with `@pytest.mark.docker`. Run them with:

```bash
cd packages/<package> && uv run pytest -m docker -s -n0
```


### Linting & Formatting
```bash
uv run pre-commit run -a
```

If type checkers (ty, basedpyright) fail with unresolved imports, you likely need all packages installed: `uv sync --all-packages --all-extras`

## Project Structure
- `packages/llama-index-workflows/src/workflows/` - Main library code
- `packages/llama-index-workflows/src/workflows/server/` - Web server implementation
- `packages/llama-index-workflows/tests/` - Test suite
- `packages/llama-agents-core/` - Shared schemas and utilities for cloud components
- `packages/llama-agents-control-plane/` - Control plane service (K8s management, deployment API)
- `packages/llama-agents-appserver/` - Application server (runs workflows in pods)
- `packages/llamactl/` - CLI tool for interacting with deployments
- `packages/llama-agents-agentcore/` - Agent runtime and deployment logic
- `operator/` - Go-based Kubernetes operator (see `operator/AGENTS.md`)
- `charts/` - Helm charts (see `charts/AGENTS.md`)
- `docker/` - Dockerfiles for container images
- `operator/tilt/` - Tilt dev environment support files
- `examples/` - Usage examples

## Architecture

See `architecture-docs/` for high-level architectural overviews:
- [`core-overview.md`](architecture-docs/core-overview.md) — Workflow, Context, Runtime, and event flow
- [`control-loop.md`](architecture-docs/control-loop.md) — The reducer-based execution engine
- [`server-architecture.md`](architecture-docs/server-architecture.md) — HTTP server, persistence, and runtime decorators
- [`overall-architecture.md`](architecture-docs/overall-architecture.md) — Cloud platform architecture (operator, control plane, appserver)
- [`build-api.md`](architecture-docs/build-api.md) — Build API for deployment creation
- [`quick-reference.md`](architecture-docs/quick-reference.md) — Code navigation guide and key constants

The DBOS package has its own architecture doc explaining the distributed model (process boundaries, adapter rules, idle release):
- [`packages/llama-agents-dbos/ARCHITECTURE.md`](packages/llama-agents-dbos/ARCHITECTURE.md)

## Key Components
- **Workflow** - Main orchestration class
- **Context** - State management across workflow steps
- **Events** - Event-driven communication between steps
- **WorkflowServer** - HTTP server for serving workflows as web services

## Versioning with Changesets

```bash
npx changeset              # Add a changeset
npx changeset status       # Check pending changes
```

Changeset descriptions should be a single line in plain English. Keep it short and simple. Do not use conventional-commit prefixes (no `fix(scope):`, `feat(scope):`, etc.).

## Development Environment

```bash
uv run operator/dev.py up             # Set up kind cluster and start development
uv run operator/dev.py down           # Clean up deployed resources
uv run operator/dev.py down --delete  # Delete the kind cluster
uv run operator/dev.py status         # Show cluster status
```

## Other Components

For Operator/Helm details, see `operator/AGENTS.md` and `charts/AGENTS.md`.

## Notes for Claude
- Always run tests after making changes: `uv run dev`
- Never use classes for tests, only use pytest functions
- Always annotate with types function arguments and return values
- The project uses async/await extensively
- Context serialization requires specific JSON format for globals

## Autonomous Operation

The following rules apply if you are running in an isolated sandbox environment and have tools to commit and push changes to git

Make sure to install uv as the package manager. Development commands rely on it.

```bash
curl -fsSL https://astral.sh/uv/install.sh | sh
```

Always run tests and pre-commit before committing:

```bash
uv run dev
uv run pre-commit run -a
```

## Testing Patterns

We use **pytest** with idiomatic pytest patterns. Follow these guidelines:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [run-llama/llama-agents](https://github.com/run-llama/llama-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
