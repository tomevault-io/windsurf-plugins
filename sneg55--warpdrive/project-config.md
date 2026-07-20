---
trigger: always_on
description: Guidance for working in this codebase (with an AI coding assistant or by hand).
---

# Project Instructions

Guidance for working in this codebase (with an AI coding assistant or by hand).

**Project:** warpdrive
**Description:** Open-source, single-tenant, self-hosted business-development CRM that reimplements Pipedrive's BD workflow: pipeline management, deal workspace, contacts/organizations, Gmail two-way email, user management, notifications, and basic stats.

## Stack

- Next.js (App Router) + React, TypeScript
- Drizzle ORM + Postgres (JSONB for custom fields)
- tRPC (typesafe reads) + TanStack Query; Next server actions for mutations/forms
- shadcn/ui + Tailwind; dnd-kit for the board; Zustand for board-local state
- Self-hosted WebSocket server + Postgres LISTEN/NOTIFY for realtime
- Gmail API via OAuth (Google Workspace); Google SSO for auth
- pg-boss for background jobs; MinIO (S3-compatible) for file storage
- pnpm

## Conventions

- Organize by feature under `src/features/`, not by technical layer. Each feature holds its implementation, types, constants, validation, and tests together.
- Keep files small (target under ~200 lines); split a single responsibility once it grows past ~300. Shared types go in `src/types/` to break import cycles; use `import type`.
- Named constants in `src/constants/`, no magic strings.
- Comments explain WHY, not what.
- No em dashes in any file. Use commas, colons, parentheses, or "vs."/"or"/"to".

## Test-Driven Development (required)

Every feature and every bugfix is written test-first. No production code before a failing test exists for it.

The loop (Red, Green, Refactor):

1. **Red:** write the smallest test that expresses the next behavior. Run it. Watch it fail for the right reason. A test that passes on first run is suspect: delete it and write one that fails first.
2. **Green:** write the minimum code to make it pass. Nothing more.
3. **Refactor:** clean up with the test as a safety net. Re-run.

Rules:

- **Bugfixes start with a failing test that reproduces the bug.** Fix only after you have seen it fail.
- **Do not mock the database.** Integration tests hit a real Postgres test database. Mock/prod divergence hides broken queries and migrations, which is exactly the layer a CRM cannot get wrong. Tests spin up a disposable Postgres (Docker) and run against the real Drizzle migrations.
- **Test runner:** Vitest. Co-locate tests inside the feature directory (`src/features/<feature>/*.test.ts`). The suite is split into a container-less `unit` project and a real-Postgres `integration` project (`pnpm test:unit` / `pnpm test:integration`).
- Never claim something works without running the test and seeing it pass. Evidence before assertions.

## Engineering Conventions

These patterns exist because they catch the failure modes an AI coder actually produces.

### Validate at the boundary (Zod)

External data passes through a Zod schema exactly once, at the point it enters the type system; inside the program the inferred type is trusted, never re-checked. Boundaries: tRPC procedure inputs, server-action arguments, external API responses and push payloads, OAuth callbacks, CSV import rows, file reads, and env vars. Schemas live in `src/schemas/` or the owning feature dir.

### Single env boundary

One module (`src/config/env.ts`) is the ONLY place `process.env` is read. It validates with Zod and throws at import time so misconfiguration fails fast. Everything else imports the typed `env` object. A lint rule bans raw `process.env` access elsewhere.

### Result types over throws

Any operation that can fail returns a discriminated union `Result<Ok, Err>` with discriminant `ok: true | false`, instead of throwing. Callers narrow with `if (!r.ok) return r`. Throw ONLY for programmer errors (invariant violations, unreachable branches); operational failures (bad input, missing record, timeout, permission denied) are values. This matters most for batch/parallel work (syncing many mailboxes, bulk operations) where one failure must not abandon the rest. `Result`, `ok()`, `err()`, and `assertNever()` live in `src/types/result.ts`. Pair with `switch-exhaustiveness-check`.

### Error ID registry

Every `AppError` carries a stable ID of the form `E_<DOMAIN>_<NNN>`, declared once in `src/constants/errorIds.ts`. Rules: one ID per distinct cause; never reuse a retired ID (mark it retired, keep it searchable); append, do not renumber; domain prefix required. `AppError(id, message, context)` is the only error thrown in app code; raw `throw new Error(...)` is lint-flagged.

### Thread AbortSignal through long operations

Any function that does I/O, waits, or runs longer than ~100ms takes `signal: AbortSignal` (required, not optional) and passes it to every callee that accepts one (fetch, DB client, external clients, inner functions). Call `signal.throwIfAborted()` after awaits that are not signal-aware. Release resources (handles, subprocesses, DB transactions) on abort. Never swallow `AbortError`. This keeps sync jobs and obsoleted requests actually cancellable rather than piling up.

### Use the design system, never reinvent (shadcn, hard rule)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sneg55/warpdrive](https://github.com/sneg55/warpdrive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
