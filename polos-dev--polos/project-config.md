---
trigger: always_on
description: Polos is the open-source runtime for AI agents. You write the agent; Polos handles sandboxes, durability, approvals, triggers, and observability.
---

# CLAUDE.md — Polos Developer Guide

Polos is the open-source runtime for AI agents. You write the agent; Polos handles sandboxes, durability, approvals, triggers, and observability.

**Docs**: https://polos.dev/docs | **Repo**: https://github.com/polos-dev/polos | **Discord**: https://discord.gg/ZAxHKMPwFG

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                   polos CLI (Rust)                       │
│        Single binary embedding orchestrator + UI        │
├──────────────────────┬──────────────────────────────────┤
│  Orchestrator (Rust) │         UI (React/TS)            │
│  Axum + Tokio        │    Vite + Tailwind + Shadcn      │
│  PostgreSQL (SQLx)   │    localhost:5173                 │
│  localhost:8080      │                                   │
├──────────────────────┴──────────────────────────────────┤
│              Worker (Python or TypeScript SDK)           │
│  Executes agents/workflows, connects to orchestrator    │
└─────────────────────────────────────────────────────────┘
```

**Orchestrator** (`orchestrator/`) — Core execution engine: durable logs, retries, scheduling, triggers, sandbox management. Rust with Axum, Tokio, SQLx (PostgreSQL).

**CLI** (`server/`) — The `polos` binary. Manages orchestrator and UI processes. Rust with Clap.

**UI** (`ui/`) — React 19 dashboard for monitoring agents, workflows, traces. Vite, TypeScript, Tailwind CSS, Shadcn/Radix UI.

**Python SDK** (`sdk/python/`) — Agent/workflow definitions, tool system, worker runtime. Pydantic, FastAPI, OpenTelemetry.

**TypeScript SDK** (`sdk/typescript/`) — Same capabilities as Python SDK. Fastify, Vercel AI SDK, OpenTelemetry.

## Repository Layout

```
orchestrator/          Rust orchestrator (Axum API server + execution engine)
  migrations/          PostgreSQL migration files (0001–0004)
  src/                 Source code (lib.rs entry point)
  tests/               Integration tests (require DB)
server/                Rust CLI binary (embeds orchestrator + UI at build time)
ui/                    React/TypeScript dashboard
  src/                 Components, pages, hooks, API client
sdk/python/            Python SDK (polos-sdk on PyPI)
  polos/               Package source
  tests/               pytest test suite
sdk/typescript/        TypeScript SDK (@polos/sdk on npm)
  src/                 Package source
  tests/               Test files (*.test.ts)
python-examples/       20+ example agent projects (Python)
typescript-examples/   20+ example agent projects (TypeScript)
create-polos-py/       Python project scaffolder (Click CLI)
create-polos-ts/       TypeScript project scaffolder (Clack prompts)
docs/                  Documentation site (polos.dev)
DESIGN/                Architecture and design documents
scripts/               Build and dev setup scripts
```

## Prerequisites

- **Rust** (latest stable) — for orchestrator and CLI
- **Node.js 18+** and **npm** — for UI and TypeScript SDK
- **Python 3.10+** — for Python SDK
- **PostgreSQL** — running locally
- **uv** (recommended) or pip — for Python dependency management

## Development Setup

### Full build from source

```bash
./scripts/dev-setup.sh            # Build everything, install to ~/.polos
./scripts/dev-setup.sh --release  # Release mode
./scripts/dev-setup.sh --skip-ui  # Skip UI build
```

This builds orchestrator, CLI, UI, and installs both SDKs. Add `~/.polos/bin` to your `PATH`.

### Database setup

```bash
createdb polos_local
createdb polos_test    # for integration tests
```

The orchestrator auto-runs migrations on startup. Config is in `orchestrator/.env`:
```
DATABASE_URL="postgres://postgres:postgres@localhost/polos_local"
TEST_DATABASE_URL="postgres://postgres:postgres@localhost/polos_test"
POLOS_LOCAL_MODE=true
POLOS_BIND_ADDRESS=127.0.0.1:8080
```

### Individual component builds

**Orchestrator:**
```bash
cd orchestrator && cargo build
cargo test           # requires PostgreSQL + TEST_DATABASE_URL
```

**CLI:**
```bash
cd server && cargo build
# Binary at server/target/debug/polos
```

**UI:**
```bash
cd ui && npm install
npm run dev          # Dev server at localhost:5173
npm run build        # Production build
npm test             # Vitest
```

**Python SDK:**
```bash
cd sdk/python
uv pip install -e ".[dev]"   # or: pip install -e ".[dev]"
pytest                        # Run tests
```

**TypeScript SDK:**
```bash
cd sdk/typescript
npm install && npm run build   # Build with tsup
npm test                       # Run tests
npm link                       # Link for local development
```

### Running locally

```bash
polos dev                  # Start orchestrator + UI + worker (hot reload)
polos run <agent>          # Interactive agent session
polos server start         # Start just the server
polos agent list           # List registered agents
polos tool list            # List available tools
polos logs <agent>         # Stream agent logs
```

- Dashboard: http://localhost:5173
- API: http://localhost:8080

## Code Quality

### Git hooks

```bash
./scripts/setup-git-hooks.sh   # Install pre-commit hooks
# or: pre-commit install
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [polos-dev/polos](https://github.com/polos-dev/polos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
