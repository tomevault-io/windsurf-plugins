---
trigger: always_on
description: Prefect is a workflow orchestration platform that coordinates and observes data pipelines. It provides a Python SDK for building workflows, a server backend for orchestration, and a web-based UI for managing and monitoring workflows.
---

Prefect is a workflow orchestration platform that coordinates and observes data pipelines. It provides a Python SDK for building workflows, a server backend for orchestration, and a web-based UI for managing and monitoring workflows.

# Guiding Principles

Your primary responsibility is to the project and its users. Every change should serve the broader user base — not just the immediate request. Be a quality gate: prefer correct, minimal, well-tested changes over fast ones.

# Before You Start

Before making code changes, read the public contribution guide in `docs/contribute/dev-contribute.mdx` so your work follows Prefect's contribution expectations, design principles, AI-use policy, testing standards, and maintainer review criteria.
For documentation-only changes, also read `docs/contribute/docs-contribute.mdx`.
For integration changes, also read `docs/contribute/contribute-integrations.mdx`.

# Directory Structure

```
prefect/
├── benches/                         # Benchmarks (CLI, flows, tasks, imports)
├── client/                          # prefect-client build: subset of src/prefect/ published as a separate PyPI package
├── compat-tests/                    # Tests for REST API compatibility with Prefect Cloud
├── Dockerfile                       # Production container image
├── docs/                            # Mintlify documentation (see docs/AGENTS.md)
├── examples/                        # Example flows (auto-published to docs)
├── integration-tests/               # End-to-end integration tests (require running server)
├── justfile                         # Task runner (just <command>)
├── load_testing/                    # Load/performance testing
├── plans/                           # Design/implementation plan documents
├── pyproject.toml                   # Root package config
├── schemas/                         # JSON schemas (prefect.yaml, settings)
├── scripts/                         # Code generation and release scripts
├── src/
│   ├── integrations/                # External service integrations (see src/integrations/AGENTS.md)
│   └── prefect/                     # Core package: SDK, server, CLI (see src/prefect/AGENTS.md)
├── tests/                           # Test suite, mirrors src/prefect/ (see tests/AGENTS.md)
├── tools/                           # Build tools
├── ui/                              # Vue UI (legacy, will be replaced by ui-v2)
├── ui-v2/                           # React UI rewrite (see ui-v2/AGENTS.md)
└── uv.lock                          # Lockfile (auto-generated, do not edit manually)
```

# Essential Commands

```bash
# Dependencies
uv sync                                # Install all dev dependencies
just install                           # Same as above, plus perf group

# Running code
uv run -s my_script.py                             # Run a script with an editable prefect install
uv run --extra aws repros/1234.py                  # Run repro needing an integration extra
uv run --project ./src/integrations/<name> <cmd>   # Run against a local integration from repo root
prefect server start                               # Start local server
prefect config view                                # Inspect current configuration

# Testing (see tests/AGENTS.md for full details)
uv run pytest tests/path.py -k name    # Run specific test
uv run pytest tests/path.py -x -n4     # Parallel, stop on first failure

# Linting & formatting
uv run ruff check --fix .              # Lint with auto-fix
uv run ruff format .                   # Format code
uv run pre-commit run --all-files      # Run all pre-commit hooks

# Docs (from repo root)
just docs                              # Start Mintlify dev server at localhost:3000
just generate-docs                     # Regenerate all doc artifacts

# UI (from repo root)
just ui-v2                             # Start React dev server at localhost:5173

# Docker
docker build -t prefect .                              # Default build (Python 3.10)
docker build --build-arg PYTHON_VERSION=3.12 -t prefect .  # Custom Python version
docker build --build-arg EXTRA_PIP_PACKAGES="prefect-aws" -t prefect .  # With extras
```

## Quick Reference

| Component | Path | Tests |
|-----------|------|-------|
| Core SDK (flows, tasks, states, deployments) | `src/prefect/` | `tests/` |
| Flow & task engines (async orchestration) | `src/prefect/flow_engine.py`, `task_engine.py` | `tests/engine/` |
| Client SDK (schemas, HTTP client) | `src/prefect/client/` | `tests/client/` |
| Server (API, database, scheduling) | `src/prefect/server/` | `tests/server/` |
| CLI | `src/prefect/cli/` | `tests/cli/` |
| Settings | `src/prefect/settings/` | `tests/settings/` |
| Integrations (`prefect-aws`, `prefect-dbt`, etc.) | `src/integrations/` | per-integration |
| React UI (replacing Vue `ui/`) | `ui-v2/` | `ui-v2/e2e/` |

## Tech Stack & Tooling

- **Python >=3.10,<3.15** with modern typing (`list[int]`, `T | None`)
- **FastAPI** for REST APIs
- **Pydantic v2** for validation
- **SQLAlchemy 2.0** async ORM
- **Alembic** for database migrations
- **PostgreSQL/SQLite** databases

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PrefectHQ/prefect](https://github.com/PrefectHQ/prefect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
