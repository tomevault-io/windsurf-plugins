---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dependencies
bun install

# Development (starts both frontend :3001 and API :3333)
bun run dev

# Build all packages
bun run build

# Run all tests
bun run test

# Lint & format (Biome)
bun run check

# Type checking
bun run typecheck

# Initialize database schema
bun run init-db:pgsql   # PostgreSQL
bun run init-db:mysql   # MySQL
bun run init-db:mssql   # SQL Server
bun run init-db:mongo   # MongoDB
```

### Running a single test (server package)

```bash
cd packages/server
bun run test                         # all tests
bun run test:watch                   # watch mode
bun run test:coverage                # with coverage
bunx vitest run src/path/to/file.test.ts  # single file
```

> **Dev ports**: Frontend (Vite) → `http://localhost:3001`, API → `http://localhost:3333`. Port 3333 also serves the static frontend build, but use 3001 during development.

## Architecture

This is a **Bun + Turbo monorepo** with these packages:

| Package | Role |
|---------|------|
| `packages/server` | Hono API server + CLI (`npx db-studio`) |
| `packages/core` | React 19 frontend (Vite, TanStack Router/Query/Table) |
| `packages/shared` | Shared types and constants |
| `packages/proxy` | Cloudflare Workers proxy (rate limiting via Upstash Redis) |
| `www` | Marketing/docs site (TanStack Start + Fumadocs, deploys to Cloudflare) |

### Server (`packages/server`)

- **CLI entry**: `src/index.ts` — uses `commander` to parse flags (`--env`, `--port`, `--database-url`, etc.)
- **Hono app**: `src/app.ts` (or wired via `src/db-manager.ts`)
- **DB abstraction**: `src/db-manager.ts` — singleton `DatabaseManager` class; exposes `getDbPool()` (PG), `getMysqlPool()` (MySQL), `getMssqlPool()` (SQL Server, async), `getMongoClient()` / `getMongoDb()` (MongoDB). DB type is auto-detected from the URL protocol.
- **DAOs**: `src/dao/*.dao.ts` (PG), `src/dao/mysql/`, `src/dao/mssql/`, `src/dao/mongo/`
- **DAO factory**: `src/dao/dao-factory.ts` — `getDaoFactory(dbType)` returns the correct DAO implementation; routes call this instead of dispatching manually.
- **Routes**: `src/routes/` — each route file uses `new Hono<RouteEnv>()` (not `AppType`) to avoid circular imports and to access `c.get("dbType")`
- **Middleware**: `src/middlewares/` — sets `c.set("dbType", ...)` based on the connection URL
- **Type mapping**: `src/utils/column.type.ts` — `mapPostgresToDataType` / `mapMysqlToDataType` → `CellVariant`

**Multi-database routing pattern**: middleware detects DB type from `DATABASE_URL` protocol and sets `c.get("dbType")`. Routes call `getDaoFactory(dbType)` to get the right DAO implementation.

### Frontend (`packages/core`)

- TanStack Router with file-based routing in `src/routes/`
- TanStack Query for server state; Zustand for client state (`src/stores/`)
- shadcn/ui components; Monaco editor for JSON/query editing
- API calls proxied from Vite dev server (`/api` → `:3333`)
- Cell rendering: `CellVariant` = `"text" | "boolean" | "number" | "enum" | "json" | "date" | "array"`

### Shared (`packages/shared`)

Three export paths:
- `shared` / `shared/types` → `src/types/index.ts`
- `shared/constants` → `src/constants/index.ts`

### Key types

- `DATABASE_TYPES = ["pg", "mysql", "mssql", "mongodb"]` in `database.types.ts`
- `RouteEnv` — Hono env type that provides `c.get("dbType")`
- `CellVariant` / `DataTypes` — used for table cell rendering

## Tooling

- **Linter/Formatter**: Biome (tabs, 95-char width). Run `bun run check` to auto-fix.
- **Tests**: Vitest (server package only). Path aliases `@` → `./src` and `shared` → `../shared/src` are configured in `vitest.config.ts`.
- **Pre-commit hook**: runs `bun run check && bun run test && bun run build` via Husky.
- **CI**: GitHub Actions on push to `stage` — build → biome check → tests.

## Conventions

- **Commit format**: `<type>(<scope>): <message>` (e.g., `feat(back): add mysql row insert`)
- **Branch format**: `<type>/<issue-number>/<description>` (e.g., `feat/123/support-mysql`)
- **PG specifics**: `$1/$2` placeholders, FK violation code `23503`
- **MySQL specifics**: backtick identifiers, `?` placeholders, no `RETURNING` clause, FK violation errno `1451`; `mysql2`'s `execute()` requires `as any` cast when passing `unknown[]` arrays
- **MSSQL specifics**: bracket identifiers (`[col]`), named `@param` placeholders via `mssql` package
- **MongoDB specifics**: no schema enforcement; `ObjectId` handling via `isValidObjectId` / `coerceObjectId` helpers in `db-manager.ts`; "tables" are collections

## Patterns

### File Naming

| Artifact | Convention | Example |
|----------|-----------|---------|
| React hook | `use-[feature].ts` | `use-databases-list.ts` |
| Component | `[feature]-[description].tsx` | `sidebar-list-tables-item.tsx` |
| Store | `[entity].store.ts` | `database.store.ts`, `queries.store.ts` |
| Server route | `[resource].routes.ts` | `tables.routes.ts`, `records.routes.ts` |
| Base DAO (PG) | `[action]-[resource].dao.ts` | `add-column.dao.ts` |
| MySQL DAO | `[action]-[resource].mysql.dao.ts` | `add-column.mysql.dao.ts` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [husamql3/db-studio](https://github.com/husamql3/db-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
