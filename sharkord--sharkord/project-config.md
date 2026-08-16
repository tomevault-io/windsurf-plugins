---
trigger: always_on
description: Guide for AI agents working on Sharkord. Read [CONTRIBUTING.md](CONTRIBUTING.md) for
---

# AGENTS.md

Guide for AI agents working on Sharkord. Read [CONTRIBUTING.md](CONTRIBUTING.md) for
project scope and PR rules — this file covers how the code is organized.

Core principle from CONTRIBUTING: **no over-engineering**. Follow the existing pattern,
add the smallest thing that works, don't introduce unneeded abstractions or dependencies for a
single use case, only if absolutely necessary. If you find yourself writing a lot of new code, check if it can be
done with existing helpers or patterns first.

## Architecture

Bun workspaces monorepo. `bun install` at the root, `./start.sh` (tmux) or `bun dev` in
each app to run.

| Workspace             | What it is                                                                     |
| --------------------- | ------------------------------------------------------------------------------ |
| `apps/server`         | Bun + tRPC + Drizzle (SQLite) + mediasoup (voice SFU)                          |
| `apps/client`         | React + Vite + Redux Toolkit + Tailwind                                        |
| `packages/shared`     | Types, enums, helpers shared by client and server. Cross-cutting types go here |
| `packages/ui`         | Presentational components only, no app logic                                   |
| `packages/plugin-sdk` | Public API surface for plugins                                                 |
| `packages/e2e`        | Playwright end-to-end tests                                                    |

Client ↔ server talk over **tRPC**: queries/mutations plus WebSocket subscriptions for
realtime events. Everything that can't be tRPC (login, uploads, static/public files,
healthz, plugin bundles) lives in `apps/server/src/http`.

Server boot order is explicit in `apps/server/src/index.ts` (dirs → db → plugins →
servers → mediasoup → voice runtimes → crons). The top imports there are order-sensitive;
don't reorder them.

Runtime state (voice rooms, mediasoup transports) lives in `src/runtimes`. Anything
persistent goes through the database.

## Backend code style & file structure

`apps/server/src`:

- `routers/<domain>/` — one tRPC procedure **per file**, named after what it does
  (`send-message.ts`, `ban.ts`). The file exports a single `<name>Route`; the domain's
  `index.ts` does nothing but import those and compose the `t.router({ ... })` map.
  Subscriptions all live in the domain's `events.ts`. New endpoint = new file, never
  append to an existing procedure file.
- `db/queries/` reads, `db/mutations/` writes, one file per table/domain. Anything reused
  by more than one route belongs here instead of being inlined.
- `db/publishers.ts` — publishes realtime events consumed by subscriptions.
- `helpers/` — domain-aware helpers (permissions, paths, file crypto, sanitizing).
  `utils/` — infrastructure with no domain knowledge (trpc setup, pubsub, env, rate
  limiters). If unsure, it's a helper.
- `queues/` — background work off the request path. `crons/` — scheduled jobs.
- `plugins/` — plugin loading, registries, event bus, sandboxing.

Conventions:

- **Named exports only.** Server files declare `const foo = ...` and export at the bottom
  (`export { foo };`); client/shared files use inline `export const`. Match the file
  you're in.
- **kebab-case filenames**, one clear responsibility per file. Files are small on purpose.
- Types are `T`-prefixed (`TJoinedMessage`), declared next to their use or in
  `packages/shared` when both sides need them. Avoid `any`.
- Every procedure validates input with `zod` and starts with permission checks
  (`ctx.needsPermission` / `ctx.needsChannelPermission`).
- Use `invariant(condition, { code, message })` from `utils/invariant` instead of throwing
  `TRPCError` by hand. Messages are user-facing — write them that way.
- Use `protectedProcedure`; wrap in `rateLimitedProcedure` for anything a user can spam,
  with limits from `config.rateLimiters`.
- Imports are auto-organized by prettier — don't hand-sort.
- **ALWAYS** use modern ES6+ syntax: `const`/`let`, arrow functions, destructuring, template literals, etc.

Run `bun run magic` (lint:fix + format + check-types) before finishing; CI enforces it.

### Don't repeat, and put code where it belongs

Before you write anything, search for it. Most of what a new route needs already exists.

- **Search first.** Grep `helpers/`, `utils/`, `db/queries/`, `db/mutations/` and
  `packages/shared` for the behaviour you are about to write. Reuse it, or extend it.
- **Two copies is the limit.** Writing the same logic a third time means it becomes a
  helper. Writing it a second time is acceptable if the two cases are genuinely unrelated.
- **Never copy-paste a route or a query and edit it.** The copy drifts from the original
  and one of the two ends up wrong. Extract what they share instead.
- **Place code by layer, not by convenience:** db access in `db/queries` / `db/mutations`,
  domain rules in `helpers/`, infrastructure in `utils/`, orchestration only in the route.
  A route that contains a raw multi-table query, or a `utils/` file that knows about
  permissions, is in the wrong place.
- **Shared means shared.** Anything both client and server rely on — constants, enums,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sharkord/sharkord](https://github.com/Sharkord/sharkord) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
