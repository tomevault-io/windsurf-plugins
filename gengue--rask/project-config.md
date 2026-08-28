---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Rask is an unofficial keyboard-first web client for ClickUp. It mirrors a ClickUp
workspace into Postgres, serves every read from that mirror, and pushes writes
back through an outbox. The browser never calls ClickUp.

`docs/architecture.md` has the reasoning and the numbers; `CONTRIBUTING.md` has
the full local setup. This file is the working subset plus the invariants that
only show up when you read several files at once.

## Commands

Bun 1.3+, not Node: internal packages export TypeScript source with no build
step, and the API and worker use Bun's native Postgres client.

```bash
bun install
bun run db:up          # Postgres on :5432 (Docker)
bun run db:migrate
bun run dev            # API :3000, Vite :5173, worker, landing :5174. Use :5173 — it proxies /api and /auth
```

```bash
bun run check          # Biome lint + format
bun run check:fix
bun run typecheck      # tsc over all five packages
bun run db:test        # (re)create the four rask_test_* databases — after any schema change
bun run test           # every package's suite
bun run e2e            # Playwright; starts its own API, Vite and database
bun run --cwd apps/web contrast   # WCAG audit of the colour tokens
```

One file, or one test:

```bash
bun run --cwd apps/api test test/filters.test.ts
bun run --cwd apps/api test test/filters.test.ts -t "rejects an unknown field"
```

Go through each package's `test` script (`bun run --cwd <pkg> test ...`), never a
bare `bun test`. The script is what sets `TEST_DATABASE_URL` to that package's
own `rask_test_*` database; without it a suite points at whatever `DATABASE_URL`
names, and these tests insert and delete real rows. A bare `bun test` at the root
also globs the Playwright specs.

Signing in locally:

```bash
bun run seed           # 450 fake tasks; writes apps/web/.dev-session
open http://localhost:5173/__dev-login
```

`seed` **deletes every row in whatever `DATABASE_URL` names** (it counts first and
refuses a database that looks real). `bun run link` instead stores
`CLICKUP_PERSONAL_TOKEN` and points the local build at your real workspace —
writes from there reach ClickUp.

## Layout

| | |
|---|---|
| [apps/web](apps/web) | SolidJS, Vite, TanStack Router + DB, Tailwind v4, CodeMirror. |
| [apps/site](apps/site) | The landing page at the apex domain. Vite + Tailwind, no framework, one static page. Shares `apps/web/src/theme.css` and nothing else. |
| [apps/api](apps/api) | Hono on Bun. Reads the mirror, writes the mirror + outbox, fans out SSE. Serves the built SPA in production. |
| [apps/worker](apps/worker) | Six self-rescheduling loops: outbox drain, webhook read-back, cold list, poll, webhook health, nightly reconcile. |
| [packages/schema](packages/schema) | Drizzle tables, idempotent ingest, token encryption. The mirror. |
| [packages/clickup-client](packages/clickup-client) | Typed ClickUp client, per-token rate limiter, vendored v2 OpenAPI spec, shared vocabulary. |

## How the pieces fit

**Reads answer from Postgres, then repair themselves.** `GET /api/tasks/:id`
returns the mirrored detail immediately and kicks off a background ClickUp
refresh; the fresher version arrives over SSE ([apps/api/src/index.ts:345](apps/api/src/index.ts:345)).
`ChangeFeed` polls `tasks.synced_at` once a second and pushes changed rows to
every stream ([apps/api/src/changes.ts](apps/api/src/changes.ts)). The one read that can block on ClickUp is
`GET /api/views/:id/tasks` — a ClickUp view's filters are ClickUp's to evaluate,
so the response is used as *membership* and the rows still come from the mirror.
It blocks only on the first open per user (and past a 24h staleness gate):
after that the walk's answer is remembered in `view_memberships`, the route
answers from it, and the fresh set follows over the `view` SSE event.

**Loading a list is what makes it worth polling.** Both task routes insert a
`sync_cursors` row for the list; the worker only ever polls lists that have one.
Nothing else registers interest.

**Writes are optimistic three layers deep.** The browser applies through the
TanStack collection ([apps/web/src/lib/store.ts](apps/web/src/lib/store.ts)); the API updates the mirror and
inserts an `outbox` row in one transaction ([apps/api/src/writes.ts](apps/api/src/writes.ts)); the worker
claims rows `FOR UPDATE SKIP LOCKED` and ships them ([apps/worker/src/outbox.ts](apps/worker/src/outbox.ts)).
A rejection is never retried forever and never papered over: the worker refetches
from ClickUp to repair the mirror, and the author gets a `write-failed` SSE event.
Rows not yet shipped carry a `tmp_` placeholder id; addressing one upstream would
404, so those writes are refused with 409 rather than queued.

**Time tracking is the one thing read live, not mirrored.** The running timer
and the individual entries come straight from ClickUp ([apps/api/src/time.ts](apps/api/src/time.ts)); the only
mirrored trace is `tasks.time_spent`, which rides free on the task payload every
sync already reads. A running timer is one row per *user* that changes only when
that person acts, so a table plus a poll loop plus a reconcile path would all

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gengue/rask](https://github.com/gengue/rask) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
