---
trigger: always_on
description: Instructions for AI coding agents working on Coop. `README.md` is for humans; this file is for machines. The nearest `AGENTS.md` to the edited file wins; explicit user prompts override everything.
---

# AGENTS.md

Instructions for AI coding agents working on Coop. `README.md` is for humans; this file is for machines. The nearest `AGENTS.md` to the edited file wins; explicit user prompts override everything.

This file inherits from the ROOST community policy — read it once:
- [ROOST community `AGENTS.md`](https://github.com/roostorg/community/blob/main/software-development-practices/agents.md) — pan-org agent rules (dependency approval, CI/CD approval, small diffs, PR standards).
- [ROOST `CONTRIBUTING.md`](https://github.com/roostorg/.github/blob/main/CONTRIBUTING.md) — contribution standards (explainable, reviewable, digestible).

## Architecture

Four independent packages, **not an npm workspace** — each has its own `package.json` and lockfile:

- `/` — root scripts, graphql-codegen, docker compose orchestration
- `/server` — Express + Apollo GraphQL API (ESM, `"type": "module"`)
- `/client` — React + Vite + Apollo Client frontend (Ant Design, TailwindCSS)
- `/db` — migration runner for Postgres, ClickHouse, Scylla
- `/migrator` — package and CLI tool for database migrations

Node **24** (`.nvmrc`). Running on Node 20 produces `EBADENGINE` warnings and can fail native builds.

Reference files: `README.md` (getting started), `server/bin/README.md` (utility scripts), `docs/` (architecture, ADRs).

## Design

- **API:** REST + GraphQL (Apollo Server); client uses Apollo Client with InMemoryCache; server resolvers live in `server/graphql/resolvers/`.
- **GraphQL authoring:** Inline in resolver files with `/* GraphQL */` comment markers — codegen discovers queries this way. Searching for `gql` or `graphql` alone misses most of it.
- **GraphQL codegen:** `npm run generate` (from root) regenerates `client/src/graphql/generated.ts` and `server/graphql/generated.ts`. **Never hand-edit** either `generated.ts`. **Never hand-merge** either `generated.ts` during a rebase/merge — pick one side with `git checkout --ours|--theirs <file>`, then run `npm run generate`. Hand-merging produces output that parses but drifts from the schema.
- **Data model:** Use Knex query builder for Postgres; ClickHouse via raw SQL in `server/clickhouse/`; Scylla via Cassandra driver.
- **Dependency injection:** Server uses BottleJS DI (wired in `server/iocContainer/`). Register services in `iocContainer`, don't export singletons from service files. Consumers receive dependencies via DI rather than importing directly. Bypassing `iocContainer` will work at runtime but breaks test mocking patterns.

## Build and run

Prerequisites: Node 24 (`.nvmrc`), Docker + Docker Compose v2, 8 GiB RAM recommended (running an instance requires 4 GiB, the rest will be used by development tools).

```bash
# Start backing services (Postgres, ClickHouse, Scylla, Redis)
npm run up

# Install dependencies in all packages
npm install
(cd client && npm install)
(cd server && npm install)
(cd db && npm install)

# Populate .env files for /server and /db, then run migrations
npm run db:update -- --env staging --db api-server-pg
npm run db:update -- --env staging --db scylla
npm run db:update -- --env staging --db clickhouse

# Create organization and admin user
npm run create-org

# Start dev servers (separate terminals recommended)
npm run client:start        # React dev server
npm run server:start        # Express + GraphQL API
npm run generate:watch      # (optional) watch GraphQL changes
```

Client: http://localhost:3001 · Server: http://localhost:3000

## Testing

Integration tests spin up services via docker compose. Unit tests run in-process.

```bash
# Run all tests (via docker compose)
docker compose run --rm test

# Server unit tests (no Docker)
(cd server && npm test)

# Client unit tests (no Docker)
(cd client && npm test)
```

Lint / format / type-check (no Docker needed):

```bash
npm run lint           # lint all packages
npm run format         # format all packages
(cd server && npm run lint)
(cd client && npm run lint)
```

If tests fail with database errors, check migration logs via `docker compose logs migrations`.

## CI

CI runs entirely via GitHub Actions (`.github/workflows/apply_pr_checks.yaml`). All PR checks are defined as `docker compose` services so you can reproduce any CI job locally. Run them in your shell (paste-as-is — each command's exit code matches the corresponding CI step's exit code):

```bash
docker compose run --rm codegen-check
docker compose run --rm backend npm run lint
docker compose run --rm backend npm run build
docker compose run --rm client npm run lint
docker compose run --rm client npm run build
docker compose run --rm test
```

Individual checks:

| CI job | Local command |
| --- | --- |
| `check_generated_graphql` | `docker compose run --rm codegen-check` |
| `check_api_server` (lint) | `docker compose run --rm backend npm run lint` |
| `check_api_server` (build) | `docker compose run --rm backend npm run build` |
| `run_frontend_checks_if_changed` (lint) | `docker compose run --rm client npm run lint` |
| `run_frontend_checks_if_changed` (build) | `docker compose run --rm client npm run build` |
| `check_api_server` (test) | `docker compose run --rm test` |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [roostorg/coop](https://github.com/roostorg/coop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
