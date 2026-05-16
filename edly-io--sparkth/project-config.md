---
trigger: always_on
description: AI-first, open-source learning platform by Edly. Provides a unified framework for course generation with integrated AI capabilities exposed via a Model Context Protocol (MCP) server.
---

# Sparkth

AI-first, open-source learning platform by Edly. Provides a unified framework for course generation with integrated AI capabilities exposed via a Model Context Protocol (MCP) server.

- REST API: `/api/` | MCP server: `/ai/mcp` | Docs: `/docs`
- Current version: `0.1.10`

## Tech Stack

**Backend:** Python 3.14, FastAPI, SQLModel (async), PostgreSQL, Redis, Alembic, FastMCP, LangChain (OpenAI/Anthropic/Google), pydantic-settings

**Frontend:** Next.js 16, React 19, TypeScript, Tailwind CSS 4, Radix UI, Bun

**Tooling:** uv (Python packages), Ruff (lint/format), mypy strict, pytest + pytest-asyncio, Docker Compose

## Key Directories

```
app/
  core/          # Settings, DB engines, security (JWT/OAuth), logger
  models/        # SQLModel DB models (base.py has TimestampedModel, SoftDeleteModel)
  api/v1/        # REST endpoints: auth, user, user-plugins, file-parser
  plugins/       # Plugin framework: base.py (SparkthPlugin, @tool), manager.py
  core_plugins/  # Built-in plugins: canvas/, openedx/, chat/, googledrive/
  mcp/           # FastMCP server, tool registration, prompts/
  services/      # Business logic layer, plugin adapters
  rag/           # Retrieval-augmented generation (loader, vectorstore, retriever)
  cli/           # Typer CLI (user management)
  migrations/    # Alembic versions

frontend/
  app/           # Next.js pages: login, register, dashboard/[pluginName]
  plugins/       # Plugin UI implementations (chat/, google-drive/)
  lib/plugins/   # Plugin system: types.ts, registry.ts, context.tsx
  components/    # Reusable UI components (settings/, ui/)

tests/           # pytest suite mirroring app structure (api/, chat/, mcp/, rag/)
.github/workflows/ # CI: lint → type-check → test on every PR
```

## Essential Commands

```bash
# Docker (recommended for full stack)
make up              # Build + start (PostgreSQL + Redis + API + frontend)
make dev.up          # Dev mode with hot reload
make down            # Stop containers
make clean           # Stop + wipe database volume

# Local backend (requires uv)
make dev             # Install dev dependencies
make api             # FastAPI on http://0.0.0.0:7727
make mcp             # MCP server (HTTP mode)
make test            # Run pytest
make cov             # Tests with coverage
make lint            # Ruff lint
make fix             # Ruff autofix + format
make mypy            # mypy --strict

# Local frontend
make frontend        # Next.js dev server on :3000
make frontend.build  # Static export → frontend/out/
make frontend.lint   # ESLint

# Database
make migrations      # Run pending Alembic migrations
make shell           # Shell inside API container
make db-shell        # PostgreSQL shell
make create-user     # Create user (pass args after --)
```

## Environment Setup

Copy `.env.example` → `.env`. Required variables:

| Variable | Purpose |
|---|---|
| `DATABASE_URL` | PostgreSQL connection string |
| `SECRET_KEY` | JWT signing key |
| `LLM_ENCRYPTION_KEY` | Fernet key for encrypting stored LLM API keys |
| `REDIS_URL` | Redis for session caching, used in chat plugin |
| `GOOGLE_CLIENT_ID/SECRET` | Google OAuth |

CI uses `DATABASE_URL=sqlite+aiosqlite:///./test.db`. Tests always run against SQLite.

## Development Workflow: Test-Driven Development (TDD)

**Always follow TDD. Write tests before implementation — no exceptions.**

### The Mandatory TDD Cycle

For every new feature, endpoint, service method, utility, or plugin tool:

1. **Write the test first** — create or update the relevant file under `tests/` mirroring the module path (e.g. `app/services/foo.py` → `tests/services/test_foo.py`)
2. **Confirm the test fails** — the test must fail before any implementation exists (red phase)
3. **Write the minimum implementation** to make the test pass (green phase)
4. **Refactor** while keeping all tests green

> Never write implementation code before a corresponding failing test exists.

For bug fixes: write a test that reproduces the bug first, verify it fails, then fix.

## Database Migrations

**Never edit an existing migration file. No exceptions.**

Any schema change — add column, drop column, rename, alter type, add index — requires a new Alembic migration file.

Editing an existing migration breaks environments that have already applied it, causing irreproducible state across dev, staging, and production.

To create a new migration, use:
```bash
alembic revision --autogenerate -m "describe your change"
```

**Never hand-craft migration filenames or revision IDs.** Always use `alembic revision --autogenerate` — it generates a valid random hex revision ID. Hand-crafted IDs risk tooling confusion and non-hex characters that break Alembic expectations.

To apply all pending migrations:
```bash
make migrations
```

### Preventing Split Heads

Multiple Alembic heads occur when two branches each generate a migration from the same parent revision and merge independently. Before creating a new migration, always check for existing heads:

```bash
alembic heads
```

If there are already multiple heads, merge them first:
```bash
alembic merge heads -m "merge migration heads"
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [edly-io/sparkth](https://github.com/edly-io/sparkth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
