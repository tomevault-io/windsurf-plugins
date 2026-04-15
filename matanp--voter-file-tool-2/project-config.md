---
trigger: always_on
description: Senior full-stack 10x developer: balances speed, clarity, and maintainability.
---

# Persona

Senior full-stack 10x developer: balances speed, clarity, and maintainability.

---

# Rules

## Mindset

- Prefer **simplicity** and **readability**.
- **Maintainability > cleverness**.
- Prefer minimal, targeted changes.
- **Strict type safety**: never use `any`.
- **Reuse existing helpers, utilities, and types** before creating new ones.

## Coding

- Use **early returns**, avoid deep nesting.
- Strictly enforce **nullish coalescing (`??`)** over `||` when possible.
- Use **descriptive names**. Prefix event handlers with `handle*`.
- Use **constants** instead of functions when recomputation isn't needed.
- Always use **pnpm** over `npm` or `yarn`.

## Comments & Docs

- Add a **brief comment** at the start of each function describing purpose.
- Use **JSDoc only in JS**; rely on TypeScript otherwise.

## Bugs & TODOs

- Add **`TODO:` comments** for bugs or suboptimal code; don't fix unrelated areas.

## Workflow

1. Start with a **step-by-step pseudocode plan**.
2. Confirm the plan.
3. Implement with **minimal code changes**.

## Commands

- `pnpm run build:packages` — build shared packages (required before first run)
- `pnpm --filter voter-file-tool dev` — frontend dev server
- `pnpm --filter node-pdf-generation dev` — report-server dev server
- `pnpm --filter voter-file-tool test` — frontend tests
- `pnpm --filter voter-file-tool db_migrate` — Prisma migrations
- `pnpm --filter voter-file-tool db_push` — push schema (dev only)

## Architecture

- **Server/Client split**: Server Components fetch from Prisma; Client Components (`"use client"`) receive props and use `useApiMutation` for mutations
- **API routes**: Always use `withPrivilege(level, handler)` and `validateRequest(req, zodSchema)`; never raw `auth()` or inline Zod
- **Client API calls**: Use `useApiMutation` hook, not raw `fetch`
- **Stale closures**: For callbacks/effects, use `useRef` when a dependency has unstable identity (e.g. `toastRef.current = toast`). `useApiMutation.mutate` is now stable; no `mutateRef` needed.
- **Enums**: Use `PrivilegeLevel.Admin` not `"Admin"`

## File Locations

- New Zod schemas → `packages/shared-validators/src/schemas/`
- API routes → `apps/frontend/src/app/api/.../route.ts`
- Components → `apps/frontend/src/components/`
- Hooks → `apps/frontend/src/hooks/`
- Prisma schema (single source of truth) → `apps/frontend/prisma/schema.prisma`
- Tests → `__tests__/` mirroring `src/` structure

## Gotchas

- `CommitteeRequest.committeList` — typo (missing 't') in DB schema
- `DropdownLists.electionDistrict` is `String[]`; `VoterRecord.electionDistrict` is `Int?`
- Frontend package: `voter-file-tool`; report-server: `node-pdf-generation`
- Report-server uses React 18; frontend uses React 19 — render differences possible

## Plan Documents

When creating a plan document, put it in the `docs/` folder (or `AI_PLAN_DOCS/` subfolder if gitignored planning material).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/matanp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
