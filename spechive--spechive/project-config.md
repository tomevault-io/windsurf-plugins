---
trigger: always_on
description: SpecHive is a multi-tenant test reporting platform that ingests test results from CI runners, stores artifacts in S3-compatible storage, and presents dashboards. Built with NestJS (backend), React/Vite (dashboard), Drizzle ORM, and PostgreSQL with row-level security.
---

# SpecHive

SpecHive is a multi-tenant test reporting platform that ingests test results from CI runners, stores artifacts in S3-compatible storage, and presents dashboards. Built with NestJS (backend), React/Vite (dashboard), Drizzle ORM, and PostgreSQL with row-level security.

## Repository structure

```
apps/
  gateway/         NestJS API gateway — JWT auth, rate limiting, proxy (port 3000)
  ingestion-api/   NestJS API — receives reporter events (port 3001, internal)
  worker/          NestJS worker — processes outbox events (port 3002, internal)
  query-api/       NestJS API — serve data to the dashboard (port 3003, internal)
  dashboard/       React/Vite SPA (port 5173)

packages/
  api-types/             Typed API response interfaces
  database/              Drizzle schema, migrations, seed, connection helpers
  eslint-config/         Shared ESLint flat config
  nestjs-common/         Shared NestJS modules (config, filters, health)
  playwright-reporter/   Playwright reporter for SpecHive
  reporter-core-protocol/ Protocol types for test reporters
  shared-types/          Branded ID types and enums (shared across all packages)
  typescript-config/     Shared tsconfig bases
```

## Key files

| File                                  | Purpose                                                     |
| ------------------------------------- | ----------------------------------------------------------- |
| `packages/database/src/connection.ts` | `setTenantContext()` for RLS, `createDbConnection()`        |
| `packages/database/src/schema/*.ts`   | Drizzle schema definitions                                  |
| `packages/database/src/migrate.ts`    | `runMigrations()` — bootstrap, outboxy DDL, Drizzle, grants |
| `test/integration-global-setup.ts`    | Seeds test data (org, project, token, user)                 |
| `commitlint.config.js`                | Conventional commit rules                                   |
| `.env.example`                        | Full environment variable reference                         |

## Prerequisites

- Node.js >= 22
- pnpm >= 9
- Docker & Docker Compose (for Postgres, MinIO, Outboxy)

## Quick start

```bash
cp .env.example .env
pnpm install
pnpm build
docker compose -f docker-compose.yml -f docker-compose.dev.yml up -d
```

## Development workflow

### Build & type checking

```bash
pnpm build          # Build all packages and apps
pnpm typecheck      # TypeScript check all packages
```

### Linting & formatting

```bash
pnpm lint           # ESLint with zero warnings
pnpm format         # Format with Prettier
pnpm format:check   # Check formatting without writing
```

### Testing

```bash
pnpm test:unit      # Unit tests (no Docker required)
pnpm test:integration # Integration tests (requires Docker)
pnpm test           # All tests (unit then integration)
```

Integration tests use `test/vitest.integration.config.ts`, run sequentially (`maxWorkers: 1`) with 30s timeout. The `globalSetup` hook seeds test data automatically.

### Database

```bash
pnpm db:generate    # Generate Drizzle migrations from schema
pnpm db:migrate     # Apply pending migrations
pnpm db:seed        # Seed database with initial data
```

### Docker

```bash
pnpm docker:up      # Start all services
pnpm docker:down    # Stop all services
```

Local development uses both compose files: `docker compose -f docker-compose.yml -f docker-compose.dev.yml up -d`

## Database architecture

### Two-role model

| Role                      | Connection string                                              | Purpose                                       |
| ------------------------- | -------------------------------------------------------------- | --------------------------------------------- |
| `spechive` (superuser)    | `postgres://spechive:spechive@localhost:5432/spechive`         | Migrations, seeding, admin ops — bypasses RLS |
| `spechive_app` (app role) | `postgres://spechive_app:spechive_app@localhost:5432/spechive` | Application queries — subject to RLS policies |
| `outboxy`                 | `postgres://outboxy:outboxy@localhost:5432/spechive`           | Outboxy transactional outbox                  |

Both app roles are created by `runMigrations()` in `packages/database/src/migrate.ts` (idempotent — safe to re-run).

### Row-Level Security (RLS)

All tenant-scoped tables have RLS policies that filter rows by `app.current_organization_id`. The application **must** set this context per transaction:

```typescript
import { setTenantContext } from '@spechive/database';

await db.transaction(async (tx) => {
  await setTenantContext(tx, organizationId);
  // All queries in this transaction are now scoped to the organization
});
```

Without `setTenantContext`, queries return zero rows (fail-closed).

### Schema hierarchy

```
organizations ─→ projects ─→ runs ─→ suites ─→ tests ─→ artifacts
                  │                               │
                  └─→ project_tokens               └─→ test_attempts

users ─→ memberships ←─ organizations

auth: refresh_tokens
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SpecHive/SpecHive](https://github.com/SpecHive/SpecHive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
