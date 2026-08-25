---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

ccpool gives a group sharing **one Claude subscription** a live, shared picture of
the account's usage and who's driving it. It is a **read-only observer plus a shared
ledger** — it never sits in the request path, never proxies, and only reads the
OAuth token Claude Code already stored.

The full pipeline is documented in the algorithm docs under
`apps/web/src/pages/docs/algorithm/` (the overview `index.md` plus observation,
attribution, views, storage-and-server, and resource-usage), published to the
marketing site at `/docs/algorithm`. `README.md` covers user-facing usage.

## Commands

```bash
pnpm install
pnpm build              # turbo build (respects ^build order; required before running the CLI)
pnpm type-check         # turbo type-check across the workspace
pnpm lint               # turbo lint (eslint)
pnpm format             # prettier --write .

pnpm test               # vitest run, on Node
pnpm test:bun           # the same suite under Bun (bun run vitest run)
pnpm vitest run packages/core/src/state/shares.test.ts   # a single file
pnpm vitest run -t "attributeShares"                     # tests matching a name

# run the built CLI
node apps/cli/dist/cli.js <command>
pnpm --filter @ccpool/cli dev <command>   # tsx, no build step

# run the built server (libSQL — a file: path or a libsql://… Turso URL)
DATABASE_URL=file:/tmp/ccpool-server.db PORT=8787 node apps/server/dist/index.js
DATABASE_URL=libsql://… CCPOOL_DB_AUTH_TOKEN=… PORT=8787 node apps/server/dist/index.js
```

Both runtimes must stay green — **CI runs the whole suite twice (Node and Bun)**. The
storage/registry contract suites and the server integration tests run on a libSQL
`:memory:` database (`file:` for the one on-disk regression test), so there is **no
external infrastructure** to provision — the whole suite is self-contained.

When manually exercising the CLI/daemon/server, isolate state with env overrides so
you don't touch real data: `CCPOOL_DIR` (ccpool's `~/.ccpool`), `CLAUDE_CONFIG_DIR`
(the Claude config dir it observes), and `CCPOOL_SERVER_URL` (points the CLI at a
dev server instead of the hardcoded host).

Note: the husky pre-commit hook only runs `lint-staged` (prettier) — it does **not**
run tests or eslint, so run `pnpm test` / `pnpm type-check` / `pnpm lint` yourself
before committing.

## Architecture

Monorepo (pnpm workspaces + turbo). The meaningful packages:

- `packages/core` — runtime-agnostic domain logic: the `Storage` interface (the one
  boundary; the adapter itself lives in storage-libsql), the `IngestSink`/`ViewSource`
  backend boundary (the storage-backed pieces the server composes + the HTTP client
  the CLI uses), the registry row/error shapes, the wire contract,
  identity/credentials, the usage poller, reset detection, the JSONL reader, the
  attribution algorithm, view assembly, and shared formatters. No UI, no process.
- `packages/storage-libsql` — the server-side backend and the **only** database code
  (`file:` local SQLite and remote `libsql://` Turso): `LibsqlDatabase` (the ONE
  client, DDL, the concrete `LibsqlRegistry`) + the `LibsqlStorage` facade. The
  server is the only thing that opens it; nothing else does.
- `packages/daemon` — the long-running observer (poll loop, lifecycle, `spawnDetached`).
- `apps/cli` — Commander + Ink CLI (the composition root). **HTTP client only — it
  never opens a database.**
- `apps/server` — the multi-tenant HTTP server (Hono; libSQL).
- `apps/web` — unrelated Astro marketing site.
- `apps/app` — a **static design mock** of the web dashboard (React/Vite, hardcoded
  data). Ships nothing; it exists only to prototype the UI. `scratch/` is the same
  kind of throwaway (a CLI design playground). Neither touches real state, and
  neither is wired into the product — don't build features on them.

### Design system (frontend and web)

strictly follow `apps/web/DESIGN.md`

### One path to the ledger, one boundary

Every machine reaches the shared ledger the **same way**: over HTTP through the
ccpool server at a hardcoded URL (`apps/cli/src/lib/links.ts#DEFAULT_SERVER_URL`,
`CCPOOL_SERVER_URL` overrides). Auth is two passwords — a shared **group password**
(proves membership) and a per-name **member password** (prevents impersonation) —
traded at init for a bearer token in a 0600 `token` file. The server
stamps every ingested row with the _authenticated_ member's name; **the CLI never
touches a database** (there is no selfhost mode, no `config.mode`).

### One machine, many accounts (profiles keyed by Claude account)

A machine can be logged into ccpool under **several Claude accounts at once** — a
personal Pro plus a shared/pooled Pro, say. Each account gets its own **profile**
under `~/.ccpool/accounts/<accountUuid>/{config.json, token, state.json}`
(`apps/cli/src/lib/config.ts`); the _live_ Claude account (whatever
`resolveAccount(resolveConfigDir())` reports) selects which profile is active.
`loadConfig()`/`saveConfig()` are **active-account-aware** — they resolve the live
account and read/write its profile — so a surface reading `loadConfig()` that

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hexxt-git/ccpool](https://github.com/hexxt-git/ccpool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
