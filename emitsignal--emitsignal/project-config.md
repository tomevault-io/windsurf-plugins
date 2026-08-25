---
trigger: always_on
description: This file is the single source of truth for agent instructions in this repository.
---

# Agent Instructions

This file is the single source of truth for agent instructions in this repository.

## What This Is

EmitSignal is a real-time notification platform. Publishers POST messages to named topics; subscribers receive them live via SSE. Emails and push notifications are dispatched asynchronously via BullMQ workers.

## Commands

**Always use `bun`, never `npm`, `yarn`, `pnpm`, or `node`.** Use `bunx` instead of `npx`. Bun auto-loads `.env` — do not add `dotenv`. Prefer Bun APIs (`Bun.serve()`, `bun:sqlite`, `Bun.file`, `Bun.$`) over third-party equivalents.

```bash
# Root (workspace-wide)
bun install            # install all dependencies
bun format             # format with Prettier (run before every commit)
bun format:check       # check formatting without writing
bun lint               # ESLint across all packages
bun lint:fix           # ESLint with auto-fix
bun test               # run all package tests

# Start everything via Docker (recommended for dev)
docker compose -f packages/emitsignal-docker/docker-compose.dev.yml up

# Server (packages/emitsignal-server)
bun run dev            # API server with --watch
bun run dev:worker     # all BullMQ workers with --watch
bun run db:migrate     # Prisma migrate dev
bun run db:seed        # seed the database
bun run db:studio      # open Prisma Studio
bun run db:generate    # regenerate Prisma client
bun test               # run server tests (Bun test runner)
bun test src/path/to/file.test.ts  # run a single test file

# Website (packages/emitsignal-website)
bun run dev            # Vite dev server on :5000
bun test               # Vitest

# Mobile (packages/emitsignal-mobile)
bun run start          # Expo dev server
bun run ios            # iOS simulator
bun run android        # Android emulator
bunx expo lint         # lint mobile package
```

## Architecture

```
Website (TanStack Start) ─┐
Mobile (Expo/RN)          ├── SSE/HTTP ──► Server (Elysia) ──► PostgreSQL
CLI                       ┘                    │                  Redis
                                               │ BullMQ queues
                                       Email / Push / Schedule workers
```

### Server (`packages/emitsignal-server`)

- **Framework:** Elysia on Bun; routes are Elysia plugins in `src/http/`
- **Source layering:** `src/lib/` is infrastructure only — singletons, side effects, and external connectors (Prisma, logger, auth, event bus, cache, queues, storage, rate limiting). `src/utils/` holds pure, stateless helpers with no Prisma/Redis/singleton dependency. `src/services/` holds application rules that may touch the database (topic, message, topic access, billing, push, transactional emails). Put new code in the narrowest of the three that fits; if a helper needs Prisma, it is a service, not a util.
- **Path alias:** `#/*` → `./src/*`. Use it for every cross-folder import; keep `./sibling` relative only within the same directory.
- **Tests:** `<module>.spec.ts` inside a `__tests__/` folder in the **same directory as the module under test** — e.g. `services/billing/get-user-plan.ts` → `services/billing/__tests__/get-user-plan.spec.ts`. Never collect specs into a catch-all `__tests__/` at the root of a subtree; that is how specs drift away from the modules they cover.
- **Database:** PostgreSQL via Prisma; schema at `prisma/schema.prisma`; generated client at `src/generated/prisma/`
- **Queues:** BullMQ backed by Redis (`src/lib/queue/`); three queues — `email`, `push`, `schedule`; workers run in `src/workers/`
- **SSE fanout:** `src/lib/event-bus.ts` — an in-process `EventEmitter` that powers the listen endpoints. Single-node only; replace with Redis pub/sub for multi-node. Transport plumbing (headers, frame encoding, heartbeat/cleanup) lives in `src/lib/sse.ts`; `GET /topics/:name/listen` and `GET /listen` are thin wrappers over one shared handler in `src/http/topic/sse-listen.ts`.
- **Auth:** Better Auth (`src/lib/auth.ts`) — magic link, passkey, API keys, optional GitHub OAuth. Auth is resolved per-request in `src/http/auth/plugin.ts`; supports cookie-based sessions (web) and `Bearer <session-token>` (mobile/CLI).
- **Rate limiting:** `rate-limiter-flexible` via Redis (`src/lib/rate-limit/`); applied globally via `src/http/plugins/rate-limit-plugin.ts`. Fails open if Redis is unavailable.
- **File storage:** provider-switched via `FILE_STORAGE_PROVIDER` env — `local` (default) or `s3` (`src/lib/storage/`)
- **Email provider:** switched via `EMAIL_PROVIDER` env — `log` (default/dev), `smtp`, or `resend`
- **Log ingestion:** switched via `LOG_INGESTION_PROVIDER` env — `stdout` (default) or `betterstack`; when a provider is set, `src/lib/logger.ts` adds a second Pino transport target alongside stdout/pretty. Falls back to stdout-only when `LOG_INGESTION_TOKEN` is missing.
- **Environment:** validated at startup by TypeBox schema in `src/schema/environment.ts`; all config accessed via the `environment` export

### Publish API


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [emitsignal/emitsignal](https://github.com/emitsignal/emitsignal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
