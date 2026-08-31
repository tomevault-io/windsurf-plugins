---
trigger: always_on
description: This file provides guidance to coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to coding agents when working with code in this repository.

## Commands

```bash
# Install dependencies
bun install

# Development (starts services through Portless)
bun run dev

# Build all packages
bun run build

# Run all tests
bun run test

# Lint & format (Biome)
bun run format

# Type checking
bun run typecheck

# Initialize database schema
bun run init-db:pgsql   # PostgreSQL
bun run init-db:mysql   # MySQL
bun run init-db:mssql   # SQL Server
bun run init-db:mongo   # MongoDB
bun run init-db:sqlite  # SQLite
bun run init-db:redis   # Redis
```

### Running a single test (server package)

```bash
cd packages/server
bun run test                         # all tests
bun run test:watch                   # watch mode
bun run test:coverage                # with coverage
bunx vitest run tests/path/to/file.test.ts  # single file
```

> **Dev URLs**: Frontend (Vite) → `https://web.db-studio.localhost`, API → `https://api.db-studio.localhost`, proxy → `https://proxy.db-studio.localhost`, docs → `https://www.db-studio.localhost`. Production-style local serving uses `https://db-studio.localhost` via the server package `start` script.

## Architecture

This is a **Bun + Turbo monorepo** with these packages:

| Package | Role |
|---------|------|
| `packages/server` | Hono API server + CLI (`npx db-studio`) |
| `packages/web` | React 19 web app (Vite, TanStack Router/Query/Table) |
| `packages/shared` | Shared types and constants |
| `packages/proxy` | Cloudflare Workers proxy (rate limiting via Upstash Redis) |
| `www` | Marketing/docs site (TanStack Start + Fumadocs, deploys to Cloudflare) |

### Server (`packages/server`)

- **CLI entry**: `src/index.ts` — uses `commander` to parse flags (`--env`, `--port`, `--database-url`, etc.)
- **Hono app**: `src/utils/create-server.ts` — creates the app, registers adapters, mounts routes, validates `/:dbType`, and serves the frontend build.
- **DB connections**: `src/db-manager.ts` owns connection creation and URL parsing; adapters import connection helpers through `src/adapters/connections.ts`.
- **Adapters**: `src/adapters/` — Strategy + Template Method architecture. PostgreSQL, MySQL, SQL Server, MongoDB, SQLite, and Redis all route through registered adapters.
- **Adapter contract**: `src/adapters/adapter.interface.ts` defines `IDbAdapter`, the single interface routes depend on.
- **Adapter registry**: `src/adapters/adapter.registry.ts` exports `adapterRegistry` and `getAdapter(dbType)`. `src/adapters/register.ts` registers each adapter before routes mount.
- **Routes**: `src/routes/` — each route file uses `new Hono<RouteEnv>()` (not `AppType`) to avoid circular imports and to access `c.get("dbType")`
- **Middleware**: `create-server.ts` validates `/:dbType` against `adapterRegistry.getSupportedTypes()` and sets `c.set("dbType", ...)`

**Multi-database routing pattern**: requests under `/:dbType/*` are validated against the adapter registry, then routes call `getAdapter(c.get("dbType"))`. Routes never branch on database type; database-specific behavior lives inside the adapter classes.

### Frontend (`packages/web`)

- TanStack Router with file-based routing in `src/routes/`
- TanStack Query for server state; Zustand for client state (`src/stores/`)
- shadcn/ui primitives live in `packages/ui`; Monaco editor for JSON/query editing
- API calls go through `src/shared/api/` — pure transport wrappers over Axios
- Cell rendering: `CellVariant` = `"text" | "boolean" | "number" | "enum" | "json" | "date" | "array"`

**Feature-first structure** — domain logic lives in `src/features/`, not flat `src/hooks/` or `src/components/`:

```txt
src/
  features/
    query-runner/   — SQL editor, query execution, result display
    schema/         — column list, add/edit column forms
    table-builder/  — create table form, foreign key builder
    tables/         — table grid, cell editing, pagination, exports
    records/        — add record form, bulk insert sheets, FK reference picker
  shared/
    api/            — endpoint functions (pure transport wrappers)
    query/          — query-key factories
  stores/           — app-wide Zustand: database, overlay registry, preferences
  hooks/            — cross-cutting hooks (databases list, rate limit, theme)
  routes/           — file-based routes; each renders a feature screen only
  components/       — shell components (sidebar, chat, command palette)
```

Each feature folder follows this shape:

```txt
features/[name]/
  components/   — feature-specific React components
  hooks/        — query + mutation hooks scoped to this feature
  stores/       — feature-local Zustand state (if any)
  screens/      — entry-point component rendered by the matching route
  index.ts      — public barrel; only export what routes or other features need
```

**Package dependency direction**:

```
web → ui
web → shared
ui  → shared (type imports only, if unavoidable)
server → shared
proxy  → shared
```

Features within `packages/web` follow the same rule internally — a feature may import
from `src/shared/`, `src/stores/`, and sibling feature `index.ts` barrels, but never
from a sibling feature's internal files.

### Shared (`packages/shared`)

Three export paths:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [husamql3/db-studio](https://github.com/husamql3/db-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
