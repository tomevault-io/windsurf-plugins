---
trigger: always_on
description: This file provides guidance to AI agents working in this repository.
---

This file provides guidance to AI agents working in this repository.

## Purpose

Kanera is an opinionated project management tool where every board inside a workspace shares the same lists and custom fields.

Hierarchy:

```text
Organisation (client)
  └─ Workspace
       └─ Board
            └─ Card
```

Durable product invariants:

- Lists are workspace-scoped, not board-scoped.
- Custom fields are workspace-scoped, not board-scoped.
- Workspace membership grants access to every board; `board_members` grants cross-organisation guest access to specific boards.
- A user belongs to exactly one organisation (`clients` row) and email is globally unique.
- Onboarding runs when `me.hasWorkspace === false`.

## Repo Shape

```text
apps/api/        Fastify + Socket.IO + Drizzle
apps/web/        Angular standalone app
packages/shared/ Shared schema, DTOs, and realtime event types
```

`@kanera/shared` is the shared source of truth:

- `@kanera/shared/schema` for Drizzle tables and inferred types.
- `@kanera/shared/dto` for Zod request and response schemas.
- `@kanera/shared/events` for Socket.IO event contracts.

API code runs as ESM via `tsx`, so relative imports in `apps/api/src` must end with `.js` even in TypeScript source.

## Commands

Run all commands from the repo root.

```bash
pnpm dev
pnpm dev:db
pnpm dev:db:down

pnpm db:generate
pnpm db:migrate
pnpm db:studio

pnpm build
pnpm lint
pnpm test:api
pnpm email:preview
pnpm test:api:integration
pnpm test:api:integration -- apps/api/src/modules/cards/routes.itest.ts
pnpm --filter @kanera/web test
pnpm --filter @kanera/web test -- card-actions-menu.popover.spec.ts
```

Notes:

- `pnpm build` and `pnpm lint` both run the TypeScript check.
- Lint output should finish with no errors, and warnings should be kept minimal.
- The API uses Node's built-in test runner via `pnpm test:api` for `*.test.ts` unit/route tests only.
- Do not pass `*.itest.ts` files to `pnpm test:api`; that does not start Postgres. Use `pnpm test:api:integration -- apps/api/src/path/to/file.itest.ts` for focused integration tests.
- API integration tests run against an isolated Docker Postgres on `localhost:55433` via `pnpm test:api:integration`, run migrations, then tear the database down.
- The web test script accepts optional spec filenames after `--`; bare filenames are matched anywhere under `apps/web`.
- The web tests are intentionally narrow and focus on realtime regression points.

## Backend Rules

Authentication and tenancy:

- JWT claims are `{ sub, cid }` where `cid` is the client id.
- Refresh tokens are stored hashed in `refresh_tokens` and sent as the `kanera_rt` httpOnly cookie scoped to `/auth`.
- Protected routes use `app.authenticate`, which populates `req.auth`.

For mutation routes, preserve this pattern:

1. Validate input with the matching DTO schema.
2. Enforce access with `assertBoardAccess(...)` or `assertWorkspaceAccess(...)`.
3. Perform the write, then call `recordActivity(...)` and emit the matching realtime event.

Do not skip the activity write or the emit. Missing either one causes broken audit or stale clients.
For board- and workspace-scoped events, the emit helper also publishes a durable `event_outbox` row. The app API drains that outbox for cross-process Socket.IO broadcasts and webhook delivery, which is how public API mutations update connected web clients near real time.

- Throw `AppError` helpers from `apps/api/src/lib/errors.ts` for API failures instead of hand-rolling error responses.

Realtime and ordering:

- Workspace-scoped events go to `workspace:${workspaceId}`.
- Board-scoped events go to `board:${boardId}`.
- Clients must explicitly join board rooms with `board:join`; workspace-scoped events should not be treated as board-local.
- Event payloads carry full entities, not diffs.
- `*:moved` events also include `prevPosition`.
- Rebalance events must be emitted before the corresponding `*:moved` event.
- Use `emitToBoard(...)` / `emitToWorkspace(...)` from route code. Do not bypass the outbox with direct Socket.IO broadcasts unless you are inside the outbox dispatcher or intentionally emitting user/client-only session events.

Positions:

- `lists.position`, `cards.position`, and `custom_fields.position` are `numeric(20,10)` stored as strings.
- Use `between(prev, next)` to assign a new position.
- If rebalancing is required, use the helpers in `apps/api/src/lib/rebalance.ts`.

## Schema Workflow

Schema changes are TypeScript-first:

1. Edit files under `packages/shared/src/schema`.
2. Re-export new schema from that package index if needed.
3. Run `pnpm db:generate`.
4. Review the generated SQL under `apps/api/drizzle`.
5. If there are pending migrations, run `pnpm db:migrate` against the dev database.
6. Commit schema and migration together.

Value domains:

- For application-owned finite string values, export one `as const` tuple, use
  `text("column", { enum: VALUES })` for Drizzle inference, and add a named PostgreSQL
  `CHECK` with `valueIn(...)` from `packages/shared/src/schema/_value-check.ts`.
- Reuse the same tuple in Zod DTOs so TypeScript, request validation, and PostgreSQL cannot drift.
- Do not introduce a native PostgreSQL enum unless the vocabulary is genuinely immutable and ordered;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [happendev/Kanera](https://github.com/happendev/Kanera) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
