---
trigger: always_on
description: - `simulation-test` is a long-running integration/fuzz harness for the `ponder` package.
---

# Simulation Test Guide

## Overview / Goal
- `simulation-test` is a long-running integration/fuzz harness for the `ponder` package.
- It runs real Ponder apps against Postgres and live chain RPCs, then verifies that the app's indexed tables match precomputed expected tables.
- The harness intentionally injects adverse conditions that normal tests do not cover: RPC errors, database errors, uncached sync gaps, realtime block delivery, reorgs, process restarts, and different event ordering modes.
- The goal is to catch regressions in sync, indexing, realtime, crash recovery, and database behavior before they reach users.

## Prerequisites
- Use the repository toolchain from the root `AGENTS.md`: pnpm `11.0.0`, Node `>=22`, and Bun for this package's scripts.
- Install workspace dependencies from the repository root with `pnpm install`.
- Build the publishable packages before running simulations with `pnpm build`; the simulation apps depend on the workspace `ponder` package.
- Provide a Postgres server connection in `DATABASE_URL`. The URL should omit the app/run database name because scripts append database names themselves, for example `postgresql://postgres@localhost:55432`.
- Provide RPC URLs used by the app under test. Current env names are `PONDER_RPC_URL_1`, `PONDER_RPC_URL_10`, `PONDER_RPC_URL_130`, and `PONDER_RPC_URL_8453`.
- Copy `simulation-test/.env.example` to `simulation-test/.env.local` for local runs, or export the variables in your shell.

## Local Postgres
- Prefer an isolated local Postgres instance for simulation-test work. Do not reuse unrelated local databases such as `ffca-postgres`.
- A known-good Docker setup is:

```bash
sudo docker run -d \
  --name simulation-test-postgres \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_DB=postgres \
  -e POSTGRES_HOST_AUTH_METHOD=trust \
  -v simulation-test-postgres-data:/var/lib/postgresql/data \
  -p 127.0.0.1:55432:5432 \
  postgres:16
```

- Use `DATABASE_URL=postgresql://postgres@localhost:55432` with this container.
- If host Postgres client tools are unavailable, use the tools inside the container with `sudo docker exec simulation-test-postgres psql ...`, `pg_dump`, or `pg_restore`.
- Verify readiness with `sudo docker exec simulation-test-postgres pg_isready -U postgres -d postgres`.

## Database Setup
- `pnpm migrate` applies the root simulation-test migrations. These create shared `metadata` and `rpc_cache` tables on the base database connection.
- `pnpm create:app [app id]` creates a Postgres database named exactly `[app id]` and runs that app once with `DATABASE_SCHEMA=expected` to populate template data.
- `pnpm test [app id]` expects a template database named `[app id]` to already exist.
- Each test run creates a UUID database with `CREATE DATABASE "<uuid>" TEMPLATE "<app id>"`, runs the simulation there, then marks the root `metadata` row as successful if validation passes.
- `src/cleanup-database.ts` drops only successful UUID databases recorded in `metadata`. Failed run databases are left behind for debugging unless manually removed.
- `pnpm create:app [app id]` is not idempotent; do not run it against shared infrastructure unless you intend to create or replace that app template database.

## Populating Local Data
- Prefer copying data from the Railway Postgres instance instead of rebuilding templates and RPC cache from live RPC. Railway has a fully synced database, and dumping it avoids expensive and rate-limited RPC backfills.
- Treat Railway as production infrastructure. Use it read-only for `pg_dump`; do not drop databases, truncate tables, run cleanup scripts, or run simulation tests against it unless explicitly asked.
- To copy a template app locally, dump the Railway app database and restore it into the isolated local Postgres instance. Use the Railway connection string from secrets or the dashboard, but change only the database name.
- Example shape for `reference-erc20`:

```bash
export LOCAL_DATABASE_URL="postgresql://postgres@localhost:55432"
export RAILWAY_SERVER_URL="postgresql://postgres:<password>@<host>:<port>"

pg_dump "$RAILWAY_SERVER_URL/reference-erc20" --format=custom --file=/tmp/reference-erc20.dump
createdb "$LOCAL_DATABASE_URL/reference-erc20"
pg_restore --no-owner --dbname="$LOCAL_DATABASE_URL/reference-erc20" /tmp/reference-erc20.dump
```

- The shared root database also contains `rpc_cache`. Copying it locally is useful when debugging cache behavior or when app templates do not already include enough sync data.
- If Railway is unreachable, the fallback is to run `pnpm create:app [app id]` locally. This repopulates template data and cache just in time from live RPC as the app runs, but it should not be the default path for large apps or routine setup.
- Verified local fallback for `reference-erc20`:

```bash
set -a; source .env.local; set +a
export DATABASE_URL="postgresql://postgres@localhost:55432"
export PGDATABASE=postgres
export PGSSLMODE=disable
pnpm migrate
pnpm create:app reference-erc20
```

## Running
- From `simulation-test`, run one fuzz simulation with `pnpm test [app id]`.
- Reproduce a specific run with `SEED=[seed] pnpm test [app id]`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ponder-sh/ponder](https://github.com/ponder-sh/ponder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
