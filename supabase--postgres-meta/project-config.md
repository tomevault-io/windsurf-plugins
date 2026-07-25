---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

postgres-meta is a RESTful API for managing PostgreSQL databases. It provides a normalized interface over the PostgreSQL system catalog, exposing database objects (tables, columns, functions, etc.) through a Fastify-based REST API. The library can be used both as a standalone server and as an npm package.

## Development Commands

### Development Server
```bash
npm run dev              # Start dev server with Docker DB (auto-cleans on exit)
npm run dev:code         # Start dev server without DB setup (if DB already running)
```

The dev server uses nodemon with ts-node/esm loader and pipes output through pino-pretty for readable logs.

### Building
```bash
npm run build           # Compile TypeScript + copy SQL files to dist/
npm run clean           # Remove dist/ and tsconfig.tsbuildinfo
npm run check           # Type-check without emitting files
```

Build process: TypeScript compilation (tsc) + copying `src/lib/sql/*.sql` files to `dist/lib/sql/` via cpy-cli.

### Testing
```bash
npm test                # Full test: db:clean -> db:run -> test:run -> db:clean
npm run test:run        # Run tests only (DB must be running)
npm run test:update     # Update test snapshots (runs db:clean -> db:run first)
```

Tests use Vitest with snapshot testing. Test files are split between `test/lib/` (library tests) and `test/server/` (API tests). All tests import from `test/index.test.ts` which orchestrates test execution.

**Important**: Tests require Docker. The test DB is managed via `test/db/docker-compose.yml` and runs on port 5432. Tests run sequentially (`maxConcurrency: 1`) and use `pool: 'forks'` to avoid memory issues.

### Database Management
```bash
npm run db:run          # Start test DB in Docker (detached, with healthcheck)
npm run db:clean        # Stop and remove test DB containers
```

### Type Generation
```bash
npm run gen:types:typescript    # Generate TypeScript types from DB schema
npm run gen:types:python        # Generate Python types (Pydantic models)
npm run gen:types:go            # Generate Go types
npm run gen:types:swift         # Generate Swift types (beta)

# With custom DB connection:
PG_META_DB_URL=postgresql://... npm run gen:types:typescript
```

Type generation is controlled by `PG_META_GENERATE_TYPES` env var and runs the server in special mode (exits after generating types to stdout).

### Code Quality
```bash
npm run format          # Format code with Prettier
```

## Architecture

### Two-Layer Design

1. **Library Layer** (`src/lib/`): Core PostgreSQL introspection logic
   - `PostgresMeta.ts`: Main class that aggregates all metadata managers
   - `PostgresMeta*.ts`: Individual managers for each database object type (columns, tables, functions, etc.)
   - `sql/*.sql.ts`: SQL query templates for fetching metadata from system catalogs
   - `Parser.ts`: SQL parsing, formatting, and deparsing (uses pgsql-parser)
   - `db.ts`: Connection pooling and query execution with error handling

2. **Server Layer** (`src/server/`): REST API built with Fastify
   - `server.ts`: Entry point, handles normal server mode + type generation mode
   - `app.ts`: Main Fastify app with routes, CORS, Swagger docs
   - `admin-app.ts`: Admin server (runs on PG_META_PORT + 1) for metrics
   - `routes/*.ts`: REST endpoints mapping to library methods
   - `templates/*.ts`: Type generation templates for different languages

### Object Manager Pattern

Each PostgreSQL object type has a dedicated manager class following this pattern:
- `PostgresMetaTables`, `PostgresMetaColumns`, `PostgresMetaFunctions`, etc.
- Each manager has methods: `list()`, `retrieve()`, `create()`, `update()`, `remove()`
- Managers compose SQL from `src/lib/sql/*.sql.ts` templates
- All methods return `PostgresMetaResult<T>` = `{ data: T | null, error: Error | null }`

### SQL Query Organization

SQL queries are defined in `src/lib/sql/*.sql.ts` as TypeScript template literals:
- Queries use `pg-format` for safe parameterization
- Complex queries join against `pg_catalog` and `information_schema`
- Build process copies SQL files to `dist/lib/sql/` for runtime access

### Connection Pooling

- Uses custom `@supabase/pg` fork (v0.0.3) instead of standard `pg`
- Connection pool initialized in `db.ts` via `init(config)`
- Pool configuration supports:
  - `connectionTimeoutMillis`: Connection timeout (default 15s)
  - `query_timeout`: Query timeout (default 55s)
  - `maxResultSize`: Max result size in bytes (default 2GB, configurable via `PG_META_MAX_RESULT_SIZE_MB`)
  - SSL configuration via `PG_META_DB_SSL_ROOT_CERT`
- Custom error handling for `RESULT_SIZE_EXCEEDED` errors
- Pool auto-reconnects on connection errors by calling `pool.end()` and reinitializing

### Error Handling

Database errors are formatted to mimic `psql` output:
- Includes severity, error code, message, line number, position marker
- Position calculation accounts for injected `SET statement_timeout` prefix
- Returns structured errors: `{ code, message, formattedError, position, detail, hint }`

### Type Generation

Type generation (`npm run gen:types:*`) works by:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [supabase/postgres-meta](https://github.com/supabase/postgres-meta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
