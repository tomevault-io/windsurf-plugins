---
trigger: always_on
description: This is a **uv workspace monorepo** with four components:
---


## Project Overview

### Workspace Structure

This is a **uv workspace monorepo** with four components:

- **`client/`** — `dhub-cli` package (open-source CLI, published to PyPI) — import path: `dhub.*`
- **`server/`** — `decision-hub-server` package (private backend, deployed on Modal) — import path: `decision_hub.*`
- **`shared/`** — `dhub-core` package (shared domain models and SKILL.md manifest parsing) — import path: `dhub_core.*`
- **`frontend/`** — React + TypeScript web UI (bundled into the server at deploy time, not a uv workspace member)

`shared/` is the single source of truth for data models (`SkillManifest`, `RuntimeConfig`, etc.) and manifest parsing. Both client and server depend on it — never duplicate these definitions.

### Tech Stack

**Backend (Python 3.11+):**
- **FastAPI** for REST API (server)
- **Typer + Rich** for CLI (client)
- **Pydantic** for data validation and settings
- **Gemini** for LLM (search, classification, gauntlet safety analysis)
- **Anthropic** for LLM (eval judging)
- **boto3** for S3 access
- **loguru** for server logging

**Frontend:**
- **React 19** with TypeScript
- **Vite** for bundling and dev server
- **React Router** for routing

**Important**: Always use `uv run` to execute Python code, not `python` directly.

## Development Setup

### Environments (Dev / Prod / Local)

The project has three environments controlled by `DHUB_ENV` (`dev` | `prod` | `local`). The server defaults to `dev` for safety; the CLI defaults to `prod` for end users.

**Always work against dev unless explicitly told to use prod.** Prefix all CLI, server, and deploy commands with `DHUB_ENV=dev`:

```bash
DHUB_ENV=dev dhub list                          # CLI against dev
DHUB_ENV=dev modal deploy modal_app.py          # deploy dev Modal app (from server/)
DHUB_ENV=dev uv run --package decision-hub-server uvicorn ...  # local dev server
```

- **Dev**: `https://hub-dev.decision.ai`, config at `~/.dhub/config.dev.json`, env file `server/.env.dev`
- **Prod**: `https://hub.decision.ai`, config at `~/.dhub/config.prod.json`, env file `server/.env.prod`
- **Local**: `http://localhost:5173`, env file `server/.env.local`, infra via `docker-compose-local.yml` (Postgres + MinIO). Requires Docker Desktop. Evals spawn to the deployed dev Modal app. When asked to deploy locally, run `make deploy-local` in the background (`run_in_background: true`).

**Working directory caveat**: Always run server-package commands from `server/`. The server's `.env.dev` / `.env.prod` files live in `server/` and `pydantic-settings` resolves them relative to the current working directory. Running from the repo root will fail with missing settings errors.

```bash
# Correct
cd server && DHUB_ENV=dev uv run --package decision-hub-server python -c "..."
cd server && DHUB_ENV=dev modal deploy modal_app.py

# Wrong — .env.dev not found
DHUB_ENV=dev uv run --package decision-hub-server python -c "..."
```

Client-package commands (`uv run --package dhub-cli ...`) can run from anywhere.

### Quick Reference

Common commands are available via `make`. Read the `Makefile` to see all targets. **Always read the `Makefile`** for available targets and their exact recipes — do not hardcode or memorize commands that the Makefile already provides.

**Important**: Always run `make` from the **repo root** (where the `Makefile` lives). If your working directory has drifted (e.g. into `server/`), use `make -C /path/to/repo-root <target>` to avoid "No rule to make target" errors.

Install pre-commit hooks once after cloning: `make install-hooks`.

## Code Standards

### Design Principles & Conventions

- **Frozen dataclasses** for immutable data models
- **Pure functions over classes** — use modules to group related functions
- **Single responsibility**: Small, single-purpose functions with one clear reason to change
- **Clear interfaces**: Descriptive names, type hints, explicit signatures — obvious inputs, outputs, and behavior
- **Domain/infrastructure separation**: Keep business logic independent from frameworks, I/O, databases. UI, persistence, and external services are replaceable adapters around a clean core
- **Testing as design**: Design for fast, focused unit tests. Pure functions and small units guide architecture
- **Readability over cleverness**: Straightforward, idiomatic Python over opaque tricks. Follow PEP 8
- **YAGNI**: No abstractions or features "just in case" — add complexity only for concrete needs
- **Continuous refactoring**: Ship the simplest thing that works, refactor as requirements evolve. Routine maintenance, not heroic effort
- **Don't worship backward compatibility**: Don't freeze bad designs to avoid breaking changes. Provide clear migration paths instead of stacking hacks
- **DRY**: Do not repeat yourself — ensure each piece of logic has a single, clear, authoritative implementation instead of being duplicated across the codebase
- **Comments**: Explain business logic, assumptions, and choices — not the code verbatim

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pymc-labs/decision-hub](https://github.com/pymc-labs/decision-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
