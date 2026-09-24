---
trigger: always_on
description: Hono backend on Bun. Responsibilities: log ingest (NDJSON + OTLP), search proxy to Quickwit, user/auth, admin operations, and API keys (ingest, personal, and service-account).
---

# apps/api — Agent Guide

## About

Hono backend on Bun. Responsibilities: log ingest (NDJSON + OTLP), search proxy to Quickwit, user/auth, admin operations, and API keys (ingest, personal, and service-account).

The frontend in `apps/web` calls this API over HTTP.

For repo-wide rules (Bun, Prettier, TS strict, tests policy), see the root `AGENTS.md`.

## Stack

- Runtime: Bun
- HTTP framework: Hono 4
- ORM: Drizzle + `pg` (PostgreSQL)
- Auth: Better Auth
- Validation: Valibot
- Quickwit client: `quickwit-js`
- Protobuf codegen: buf + `@bufbuild/protoc-gen-es`

## Run, Build, Check

```bash
bun --filter api dev         # hot-reload via `bun --hot`
bun --filter api build       # bundle to dist/
bun --filter api start       # run dist/app.js
bun --filter api check       # tsc --noEmit
bun --filter api lint        # oxlint
```

Root convenience: `bun run dev:api`, `bun run build:api`, `bun run start:api`.

## Source Layout

| Path               | Purpose                                                                                                                                                                                                                                                 |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `src/app.ts`       | Hono app composition, middleware mount, error handler, SPA static serving, boot (`main()`)                                                                                                                                                              |
| `src/config.ts`    | Resolved runtime config object                                                                                                                                                                                                                          |
| `src/env.ts`       | Hono context types: `AppEnv` (`requestId`, `session?`, `apiKey?`), `AuthedEnv`, `KeyedEnv`                                                                                                                                                              |
| `src/types.ts`     | Public, pure-type surface re-exported via `exports['./types']`                                                                                                                                                                                          |
| `src/index.ts`     | Workspace package entry                                                                                                                                                                                                                                 |
| `src/routes/`      | One file per resource (`api-keys`, `auth`, `exports`, `health`, `indexes`, `monitoring`, `service-accounts`, `settings`, `shares`, `traces`, `users`, `views`); `routes/admin/` (`activity`, `cluster`, `metrics`); `routes/ingest/` (`ndjson`, `otlp`) |
| `src/services/`    | `*.service.ts` — business logic called from routes                                                                                                                                                                                                      |
| `src/middleware/`  | `request-context`, `require-user`, `require-admin`, `require-api-key`, `require-user-or-personal-key`, `with-index-config`, `with-index-meta`                                                                                                           |
| `src/schemas/`     | Valibot request schemas per resource; response schemas under `schemas/responses/`                                                                                                                                                                       |
| `src/db/`          | Drizzle schemas (`schema.ts`, `auth.schema.ts`) and DB client (`index.ts`)                                                                                                                                                                              |
| `src/lib/`         | Cross-cutting clients and IO: `auth`, `db`, `quickwit`, `quickwit-proxy`, `quickwit-metrics`, `secret`, `openapi/`, `query/`                                                                                                                            |
| `src/utils/`       | Pure helpers: `http-error`, `quickwit-error`, `otlp-response`, `bearer`, `params`, `valibot`, `require-env`, `db`                                                                                                                                       |
| `src/gen/`         | Generated protobuf code — **do not edit**                                                                                                                                                                                                               |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rootprint/rootprint](https://github.com/rootprint/rootprint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
