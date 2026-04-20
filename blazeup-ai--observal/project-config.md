---
trigger: always_on
description: Internal context for contributors and AI coding agents. Use `README.md` as the public source of truth for API endpoints and CLI usage. Use `SETUP.md` for environment setup.
---

# AGENTS.md

Internal context for contributors and AI coding agents. Use `README.md` as the public source of truth for API endpoints and CLI usage. Use `SETUP.md` for environment setup.

## Current state

Observal is an agent-centric registry and observability platform for AI coding agents. Agents are the primary entity, bundling 5 component types: MCP servers, skills, hooks, prompts, and sandboxes. All components have CRUD, CLI commands, admin review, feedback, and telemetry collection. Agents bundle components via a polymorphic junction table (`agent_components`).

All API routes accept either UUID or name for path parameters. Admin review controls public registry visibility only. Submitters can install and use their own items immediately without approval.

The MCP validator supports multiple frameworks: FastMCP, standard MCP SDK (Python), TypeScript SDK (`@modelcontextprotocol/sdk`), and Go SDK (`mcp-go`). There is no framework enforcement - all MCP implementations are accepted.

The web frontend is a Next.js 16 / React 19 app in `web/`. It uses three route groups: `(auth)` for login, `(registry)` for the public-facing agent browser, component library, and agent builder, and `(admin)` for the admin dashboard, traces, eval, review, and user management. The frontend has a custom OKLCH design system with 5 themes (light, dark, midnight, forest, sunset), three typefaces (Archivo for display, Albert Sans for body, JetBrains Mono for code), and a 4pt spacing scale. It uses shadcn/ui components, Recharts for charts, TanStack Query for data fetching, and TanStack Table for sortable/filterable tables. Shared API response types live in `web/src/lib/types.ts`. The GraphQL API at `/api/v1/graphql` is the read layer for telemetry data; REST endpoints serve everything else.

## CLI structure

The CLI is organized into nested command groups:

```
observal
├── pull                     # install complete agent (primary workflow)
├── scan                     # detect and instrument IDE configs
├── use / profile            # swap IDE configs from git-hosted profiles
├── auth                     # init, login, logout, whoami, status
├── config                   # show, set, path, alias, aliases
├── registry                 # component registry parent group
│   ├── mcp                  #   submit, list, show, install, delete
│   ├── skill                #   submit, list, show, install, delete
│   ├── hook                 #   submit, list, show, install, delete
│   ├── prompt               #   submit, list, show, install, render, delete
│   └── sandbox              #   submit, list, show, install, delete
├── agent                    # create, list, show, install, delete, init, add, build, publish
├── ops                      # observability commands
│   ├── overview, metrics, top, traces, spans
│   ├── rate, feedback
│   └── telemetry            #   status, test
├── admin                    # admin commands
│   ├── settings, set, users
│   ├── penalties, penalty-set, weights, weight-set
│   ├── review               #   list, show, approve, reject
│   └── eval                 #   run, scorecards, show, compare, aggregate
├── self                     # upgrade, downgrade
└── doctor                   # diagnose IDE settings compatibility
```

Deprecated root-level aliases exist for backward compatibility (e.g. `observal submit` → `observal registry mcp submit`, `observal upgrade` → `observal self upgrade`). These are hidden from `--help` and print deprecation warnings.

## Commands

```bash
# Docker stack (7 services: api, db, clickhouse, redis, worker, web, otel-collector)
make up                  # start
make down                # stop
make rebuild             # rebuild and restart
make logs                # tail logs

# CLI (installed via uv)
uv tool install --editable .
observal auth login      # auto-creates admin on fresh server, or login
observal auth whoami     # check auth
observal auth status     # server health check

# Linting
make lint                # ruff check
make format              # ruff format + ruff fix
make check               # pre-commit on all files
make hooks               # install pre-commit hooks

# Tests (526 tests across 18 files, run from observal-server/)
make test                # quick
make test-v              # verbose
# or manually:
cd observal-server && uv run --with pytest --with pytest-asyncio --with pyyaml --with typer --with rich --with docker pytest ../tests/ -q
```

## Important files

### API Server (`observal-server/`)

- `main.py` : FastAPI app entrypoint; mounts REST routes + GraphQL at `/api/v1/graphql`; middleware stack includes CORS (env-var origins), security headers, request size limit, rate limiting (slowapi), and session management
- `config.py` : pydantic-settings: DATABASE_URL, CLICKHOUSE_URL, REDIS_URL, SECRET_KEY, eval model config, OAuth settings (all default to None/disabled), rate limit settings
- `worker.py` : arq WorkerSettings; background eval jobs consume from Redis queue
- `api/deps.py` : auth dependency (`get_current_user` via X-API-Key header), DB session injection, `resolve_listing` (name-or-UUID resolver used by all routes)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BlazeUp-AI/Observal](https://github.com/BlazeUp-AI/Observal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
