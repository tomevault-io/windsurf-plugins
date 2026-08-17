---
trigger: always_on
description: Guidance for agents in this repo. Shared stack conventions are imported above; everything below is kinora-specific and overrides the profile where they differ.
---

# kinora

Guidance for agents in this repo. Shared stack conventions are imported above; everything below is kinora-specific and overrides the profile where they differ.

## What this is

kinora is a dashboard for Playwright test reports across projects and over time, with an embedded trace viewer. CI runs push results to a self-hosted kinora server; the dashboard tracks pass rates, trends, and flaky tests, and opens the full Playwright trace inline for failures.

pnpm workspace monorepo. Node 26, ESM-only (except `desktop`, see below), TypeScript strict. Fair source: `server`, `web`, and `desktop` are FSL-1.1-MIT (source-available, converts to MIT after 2 years); the embeddable libs (`reporter`, `cli`, `core`, `ui`, `mcp`) and `trace-viewer` are MIT.

`packages/desktop` is an Electron app: a local Playwright trace viewer (no account) plus an account dashboard that signs in to a kinora server. See the Desktop app section below.

## Commands

Run from the repo root unless noted.

```bash
pnpm install
pnpm build         # pnpm -r build, every package (tsdown for libs/server, vite for web/viewer)
pnpm typecheck     # tsc / vue-tsc across the workspace
pnpm lint          # eslint . (lint:fix to autofix)
pnpm test          # pnpm -r test (vitest). NOTE: server has no `test` script, so this skips it
pnpm test:integration  # pnpm -r test:integration; only @kinora/server has it (needs Postgres on :5436)
pnpm test:e2e      # pnpm -r test:e2e (trace viewer + web); --filter @kinora/web test:e2e to scope
                   # web e2e self-boots server+web via Playwright `webServer` (packages/web/playwright.config.ts)
                   # on dedicated ports against the `kinora_e2e` DB; only needs Postgres up on :5436

pnpm dev:server    # @kinora/server on :3000 (tsx watch)
pnpm dev:web       # dashboard on :5173
pnpm dev:viewer    # trace viewer on :5174
```

Desktop (from `packages/desktop`; build the viewer once first - the app serves its `dist/`):

```bash
pnpm --filter @kinora/trace-viewer build  # one-time, from repo root
pnpm dev                          # build main (cjs) + home UI, launch Electron
pnpm start path/to/trace.zip      # open a specific local trace
pnpm probe                        # headless self-check, exits 0/1 (VIEWER; HOME via probe:home)
pnpm dist:mac                     # local package (dmg + zip). The signed+notarized release is the Desktop Release CI workflow
```

CI (`.github/workflows/ci.yml`) is three jobs: `check` (lint -> typecheck -> build -> test), `integration_tests` (Postgres service + `@kinora/server test:integration`), and `e2e_tests` (Postgres service; `pnpm test:e2e` self-boots the stack on dedicated ports and resets `kinora_e2e` via `db:reset:e2e`, runs both the viewer and web suites). Build is in CI because cross-package types resolve through each lib's build output for published packages.

Single test / package-scoped:

```bash
pnpm --filter @kinora/core test                      # one package's vitest suite
pnpm --filter @kinora/core exec vitest run -t "name" # one test by name
pnpm --filter @kinora/server test:integration        # server suite (Postgres must be up)
pnpm --filter @kinora/server typecheck               # one package's typecheck
```

Server database (from `packages/server`, needs `.env` copied from `.env.example` and `docker compose up -d` against `packages/server/docker-compose.yml` for Postgres on :5436):

```bash
pnpm migrate           # apply pending migrations (alias: `pnpm migrate latest`); knex CLI args pass through
pnpm db:create         # create the database
pnpm db:seed           # seed demo account + data, prints login (demo@kinora.dev / password123) + an API token
pnpm db:seed:market    # larger "marketing" seed dataset
pnpm db:reset:e2e      # drop + recreate `kinora_e2e` (used by web e2e)
pnpm purge-expired-runs # retention sweep: delete runs past their retention window
```

Migrations are **knex**, not drizzle: hand-written `.ts` files in `packages/server/migrations/` (timestamp-prefixed), run by `scripts/migrate.ts`. Drizzle is the query/ORM layer only - there is no `drizzle.config.ts` and no schema-push/generate flow. To change the schema: edit `src/db/schemas/`, then add a matching knex migration. The migrate config reads connection details from `src/lib/env.ts`; in dev it loads `.ts` migrations, in prod the build emits them as `.mjs` in `dist/`.

## Architecture

### Ingest data flow

A test result travels: **Playwright run -> reporter or CLI -> `@kinora/core` normalize -> POST `/api/v1/runs` -> Postgres -> tRPC `dashboard` router -> `@kinora/web`**, with trace.zip artifacts on a parallel path.

- `@kinora/core` is the shared contract layer. zod schemas in `src/contracts/` (`kinora.ts` = stored/dashboard shapes, `ingest.ts` = the wire payload, `playwright.ts` = raw report shape) plus pure helpers in `src/lib/` (`normalize`, `aggregate`, `history`, `compare`, `status`, `test-key`). Both ingest paths and the server depend on it, so a test keeps a stable identity regardless of how it was uploaded.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kinora-dev/kinora](https://github.com/kinora-dev/kinora) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
