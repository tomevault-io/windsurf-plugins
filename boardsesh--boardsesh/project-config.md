---
trigger: always_on
description: Guidance for Claude Code working in this repo.
---

# CLAUDE.md

Guidance for Claude Code working in this repo.

## Project Overview

Boardsesh is a monorepo. Next.js 16 web app + React Native (Expo) mobile app for controlling Kilter / Tension / MoonBoard climbing boards. Adds queue management and real-time collaborative control on top of Aurora Climbing's software.

## Project Rules

- Backend work belongs in `packages/backend` (GraphQL), not in the Next.js app. We are slowly moving REST/server logic out of `packages/web`.
- Work autonomously end-to-end: backend + frontend + QA. Don't stop at "API is ready but UI isn't updated."
- Use subagents (always Opus) for grunt work. Pair every implementation subagent with a QA/reviewer subagent.
- No AI-generated images. Real photos or diagrams only.
- No buzzwords. Concrete numbers, plain language.
- Default to action. Full autonomy except no data deletion without asking.

## Monorepo Structure

```
/packages/
  /web/             # Next.js web application
  /mobile/          # React Native (Expo) mobile application
  /backend/         # WebSocket backend for party mode (graphql-ws)
  /shared-schema/   # Shared GraphQL schema and TypeScript types
  /shared/
    /play-view/     # Play-drawer logic (queue nav, tick utils, grade display)
    /queue/         # Queue state machine (reducer, types, event utils)
    /board-config/  # Board metadata, hold maps, angle tables
    /board-constants/ # Grade colours, difficulty bands
    /ble-protocol/  # Bluetooth LED control protocol
  /db/              # Shared database schema, client, migrations (drizzle)
```

## Shared Packages (Web ↔ Mobile)

Code reuse between web and mobile is the highest priority when adding cross-platform features. Before writing platform-specific logic, check whether the same behaviour already exists on the other side and extract the shared part into `packages/shared/`.

- **One responsibility per package.** Name packages after what they do (`@boardsesh/queue`, `@boardsesh/play-view`). No mega-`@boardsesh/shared`.
- **Default to pure TS; renderer-agnostic React goes in dedicated `*-react` packages.** Most shared packages stay pure TS (types, pure functions, constants, state machines) with no React at all. When web and mobile would genuinely share React hooks / context / reducers, put them in a `@boardsesh/*-react` package (e.g. `@boardsesh/queue-react`, `@boardsesh/board-react`) that lists `react` as a `peerDependency`. Such packages must stay renderer-agnostic — **no** `react-dom`, `next`, DOM globals, or MUI (web); **no** `react-native` host components or Expo APIs (mobile). Inject every platform I/O (GraphQL clients, storage, navigation, toasts) as parameters.
- **No circular deps.** Shared packages may depend on other shared packages, never on `web`, `mobile`, or `backend`.
- **Extract, don't duplicate.** When porting a web feature to mobile, pull the business logic into a shared package and update web to import from it in the same PR.
- **Tests live next to the code** in `src/__tests__/`.

## Commands

Toolchain is [Vite+](https://viteplus.dev) (`vp`). **Never use `bun run`, `bunx`, or `npx` for validation** — they bypass the unified config and can mutate `bun.lock`. The only sanctioned non-`vp` invocations are `bunx drizzle-kit generate` (migrations) and `bun run backend:start` (prod backend).

Use `vp check` or `vp run typecheck` to validate — **not** `vp run build` (build interferes with the dev server). `vp check` runs lint + format (the staged pre-commit hook calls `vp check --fix`). Run `vp run typecheck` before pushing.

Common commands:

- `vp check` — format + lint (canonical validation; pre-commit)
- `vp test` / `vp test run --reporter=agent` — tests (always use `--reporter=agent` to save context)
- `vp test --project web|backend|mobile` — scope tests
- `vp run dev` — start DB + backend + web
- `vp run dev:mobile` — start mobile dev server (Metro)
- `vp run db:up` / `vp run db:migrate` / `vp run db:studio`
- `vp run build`, `vp run typecheck` (+ `:web`, `:backend`, `:mobile`, `:db`, `:shared`)
- `vp run check:i18n` — fails on hardcoded English strings under `packages/web/app/`
- `vp run check:mobile-bundle` — headless Metro bundle check (Linux-safe)
- `vp run check:mobile-simulator`, `vp run mobile:screenshot` — macOS only
- `vp run mobile:publish` — EAS Update for current branch
- `vp run test:e2e` — Playwright; auto-starts the dev DB + web server

### Database

- `bunx drizzle-kit generate` from `packages/db/` to create migrations. **Never hand-write migration SQL** — it must be in `_journal.json`, which `drizzle-kit generate` updates for you.
- Dev DB is a pre-built image (`ghcr.io/boardsesh/boardsesh-dev-db`) with all board data, a test user (`test@boardsesh.com` / `test`), and seed data. Reset: `docker compose down -v && vp run db:up`.

### Database hosting (Railway)

We host Postgres on Railway but treat it as portable — anything we write should run on a `docker run postgres:17`. No Railway-specific addons, env vars, build steps, or schema mutations via dashboard. `pg_dump`/`pg_restore` must be sufficient to lift-and-shift. Same rule for object storage / video / analytics: prefer S3-compatible APIs, OpenTelemetry exporters, standard connection strings. Exit runbook: `docs/neon-migration.md`.

## GitHub Issue Fix Workflow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [boardsesh/boardsesh](https://github.com/boardsesh/boardsesh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
