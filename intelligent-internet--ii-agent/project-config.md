---
trigger: always_on
description: This file is the entry point for agents working in this repository. It provides the architecture overview plus a map to deeper documentation.
---

# II-Agent Contributor Guide

This file is the entry point for agents working in this repository. It provides the architecture overview plus a map to deeper documentation.

**Read `CLAUDE.md` for the full development reference** (patterns, billing system, dependency injection, code examples).

## Quick Start

```bash
uv sync --frozen          # Install dependencies
./scripts/start.sh        # Start the server
curl localhost:8000/health # Verify
```

## Repository Map

| Resource | What it covers |
|----------|---------------|
| [`CLAUDE.md`](CLAUDE.md) | Full development guide: patterns, billing, dependency injection, code examples |
| [`docs/DESIGN.md`](docs/DESIGN.md) | Design patterns: service pattern, Dep aliases, domain module structure |
| [`docs/PLANS.md`](docs/PLANS.md) | How to write and maintain execution plans |
| [`docs/RELIABILITY.md`](docs/RELIABILITY.md) | Billing reliability, cron recovery, Redis fallbacks |
| [`docs/SECURITY.md`](docs/SECURITY.md) | Auth flow (OAuth, JWT, API keys), secrets management |
| [`docs/FRONTEND.md`](docs/FRONTEND.md) | Socket.IO events, REST API surface, real-time event flow |
| [`docs/PRODUCT_SENSE.md`](docs/PRODUCT_SENSE.md) | Product principles, user personas, key journeys |
| [`docs/QUALITY_SCORE.md`](docs/QUALITY_SCORE.md) | Per-domain quality grades and health metrics |
| [`docs/design-docs/`](docs/design-docs/index.md) | Indexed design decisions with verification status |
| [`docs/exec-plans/`](docs/exec-plans/) | Active and completed execution plans |
| [`docs/generated/db-schema.md`](docs/generated/db-schema.md) | Database schema reference (from SQLAlchemy models) |
| [`docs/product-specs/`](docs/product-specs/index.md) | Product specifications |
| [`docs/references/`](docs/references/) | LLM-optimized reference material for key dependencies |

## Mandatory Rules

1. **Use `uv run`** for all Python commands (`uv run pytest`, `uv run python ...`).
2. **Run Ruff only on the Python files you changed** before marking work complete (`uv run ruff check --fix-only <changed_python_files>`, `uv run ruff format <changed_python_files>`, `uv run ruff check <changed_python_files>`, `uv run ruff format --check <changed_python_files>`). If you changed no Python files, skip Ruff.
3. **Never call `CreditService.deduct()` directly** for LLM/tool billing — use the reservation system (see `CLAUDE.md` Billing section).
4. **Use Dep aliases everywhere** — never bare `= Depends(get_x)` in function signatures (see `CLAUDE.md` Dependency Injection Pattern).
5. **Use `LLMExecutionService`** for any new code that calls an LLM outside the agent runtime loop.

## Architecture

### Domain Map

II-Agent is organized into domain modules under `src/ii_agent/`. Each domain owns its models, repository, service, dependencies, router, and schemas.

```text
src/ii_agent/
├── app/                    # FastAPI bootstrap package, middleware, lifespan, router wiring
│
├── core/                   # Shared infrastructure (no business logic)
│   ├── config/             # Pydantic settings (database, redis, storage, oauth, stripe)
│   ├── db/                 # SQLAlchemy 2.0 base, async session management, migrations
│   ├── llm/                # LLM billing service, execution service, base client
│   ├── redis/              # Redis client, cache, pubsub, lock, cancel management
│   ├── secrets/            # GCP Secret Manager integration
│   ├── storage/            # File storage abstraction (GCS, local)
│   ├── container.py        # ServiceContainer for complex dependency graphs
│   └── dependencies.py     # DBSession, SettingsDep (shared Dep aliases)
│
├── auth/                   # Authentication & authorization
│   ├── jwt_handler.py      # JWT creation/verification (HS256)
│   ├── oidc_verify.py      # OIDC token verification (RS/ES families)
│   ├── api_key_utils.py    # API key generation
│   └── users/              # User profiles, CRUD, waitlist
│
├── billing/                # Credit ledger & billing
│   ├── credits/            # Balance, ledger, pricing, service
│   ├── reservations/       # Reserve -> settle -> release state machine
│   ├── usage/              # Usage records, LLM/tool invocation telemetry
│   ├── customers/          # Stripe customer management
│   └── webhook_handler.py  # Stripe webhook processing
│
├── sessions/               # Chat session management
│   ├── models.py           # Session model, SessionStateEnum, AppKind
│   ├── service.py          # Session CRUD, state transitions
│   ├── fork_service.py     # Session forking
│   ├── title_service.py    # Auto-title generation
│   ├── wishlist/           # Session bookmarks
│   └── pin/                # Pinned sessions
│
├── agent/                  # Agent execution framework
│   ├── application/        # Validation, execution orchestration
│   ├── runtime/            # Agent runtime models, streaming
│   ├── runs/               # Agent run task management
│   ├── events/             # Event handling & logging
│   ├── prompts/            # System prompts & templates
│   ├── sandboxes/          # E2B sandbox management
│   ├── socket/             # Socket.IO command handlers (query, cancel, plan)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Intelligent-Internet/ii-agent](https://github.com/Intelligent-Internet/ii-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
