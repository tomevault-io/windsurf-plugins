---
trigger: always_on
description: Guidance for Claude Code working in this repo. For the full picture read
---

# CLAUDE.md

Guidance for Claude Code working in this repo. For the full picture read
[`docs/architecture.md`](docs/architecture.md) and [`CONTRIBUTING.md`](CONTRIBUTING.md);
this file is the short version plus the things that will trip you up.

## What this is

Minecraft Server Manager is a **single-process, server-rendered Node.js control panel** for
Minecraft servers that run as Docker containers (the `itzg/docker-minecraft-server` image). It
talks to the Docker daemon over its API via **dockerode** — never by shelling out to the `docker`
CLI. All persistent state lives under one directory (`$DATA_DIR`, default `./data`): the SQLite DB,
per-server world data, backups, the mod library. Copying that directory migrates the whole panel.

Requires **Node.js 24+** (for the flagless built-in `node:sqlite`). Package manager is **pnpm**.

## Commands

```bash
pnpm install
pnpm run dev            # app with --watch auto-restart + Tailwind CSS watch; serves raw public/js
pnpm start              # production entry (src/server.js)

# CI gates — all five must pass before a PR (run on a clean clone, no Docker/app needed):
pnpm run lint           # ESLint — errors only, no warnings tolerated
pnpm run format:check   # Prettier   (pnpm run format to fix)
pnpm run typecheck      # tsc --checkJs over the type-clean core
pnpm test               # node:test unit suite, fast, no Docker
pnpm run build          # Tailwind CSS + esbuild client-JS bundle

pnpm run test:watch     # re-run unit suite on save while iterating
pnpm run test:smoke     # scripts/qa-sweep.js — live sweep against a RUNNING panel (needs QA_USER/QA_PASS)
node --test test/foo.test.js   # run one test file
pnpm run db:migrate     # apply src/db/migrations/* by hand
```

`main` is protected; every change lands through a PR. The required `quality` status check is an
aggregate of the `checks` (lint, format, typecheck, build), `tests`, and `docker-build` CI jobs.

## Architecture — layering flows one direction only

```
web/routes/  (HTTP: parse + zod-validate input, shape responses — NO business logic)
     ↓
services/    (domain logic — the actual features; may call each other + infra)
     ↓
docker/  ·  db/  ·  storage/     (infrastructure)
```

- **`src/web/routes/`** — one Express router per domain (`servers`, `players`, `worlds`, `crashes`,
  `blueprints`, `files`, …), mounted in [`src/web/app.js`](src/web/app.js). Two routers mount in the
  **public zone** before `requireAuth`: `routes/status.js` (opt-in per-server HTML status pages) and
  `routes/apiV1.js` (`/api/v1`, read-only JSON, Bearer-token auth, off by default).
- **`src/services/`** — the heart of the app; each service owns one domain.
- **`src/docker/`** — dockerode wrappers: `connect` (endpoint auto-detected per-OS), `containers`,
  `logs`, `stats`, `images`, and `watcher` (turns Docker events into history + crash detection).
- **`src/db/`** — [`src/db/index.js`](src/db/index.js) is the **only** module that touches the
  driver (`node:sqlite`, synchronous, WAL, prepared-statement cache keyed on SQL text). API:
  `run / get / all / exec / transaction(fn) / backupTo`. Schema changes = a new numbered file in
  `src/db/migrations/`, applied on boot.
- **`src/storage/`** — the `./data` bootstrap, the **path guard** (`safeJoin`), and the background
  size-indexer + disk-quota enforcement.

Cross-cutting: **`src/config/`** (env config + the field catalog, below); **`src/events/`**
(`recordEvent()` is the one entry point for the history log); **`src/ws/`** (authenticated,
per-server **brokered** console + stats WebSockets — one upstream `docker logs --follow` per server
fanned out to every tab); **`src/logger.js`** + **`src/instrument.js`** (Pino + a dormant Sentry seam).

Boot sequence, key domain behaviors (modpacks are always pinned, the custom-mod overlay, port
allocation, disk quotas, at-rest secret encryption), and wire formats are all detailed in
[`docs/architecture.md`](docs/architecture.md).

## Conventions that will surprise you

1. **Never touch the filesystem under `./data` directly.** Resolve every path through the path guard
   in `src/storage/` (`safeJoin` / `dataPath`). It rejects anything escaping the data root — this is
   the backbone of the file-safety story. Uploads and archive extraction are additionally size-capped.
2. **Mid-function `require()` calls are intentional cycle-breakers.** If you see
   `const x = require('...')` inside a function body, it's avoiding a circular dependency at load
   time. Don't hoist it to the top without checking for the cycle.
3. **The field catalog is the single source of truth for server settings.**
   [`src/config/field-catalog/`](src/config/field-catalog/) catalogs every itzg env var / Docker
   limit / `server.properties` key with its label, help, type, default, validation, section, and
   danger flags. The wizard, settings forms, and zod validation all derive from it — exposing a new
   setting is a data change, not new UI plumbing.
4. **Server code is plain CommonJS JS — no TypeScript compile step.** Type safety is JSDoc + a
   `tsc --checkJs` gate. `types/globals.d.ts` holds ambient augmentations; dynamic-interop files
   (Docker/NBT/HTTP-JSON) carry a `// @ts-nocheck` header while typing is grown incrementally. New

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anefzaoui/minecraft-server-manager](https://github.com/anefzaoui/minecraft-server-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
