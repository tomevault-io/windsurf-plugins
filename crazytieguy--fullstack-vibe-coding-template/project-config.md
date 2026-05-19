---
trigger: always_on
description: Always follow the guidelines in this file, unless explicitly told otherwise by the user or overided in the CLAUDE.local.md file.
---

Always follow the guidelines in this file, unless explicitly told otherwise by the user or overided in the CLAUDE.local.md file.

## Project Overview

- Full-stack TypeScript app: React + Vite + TanStack Router (frontend), Convex (backend), Clerk (auth)
- Development: Run `pnpm dev` with the `run_in_background` parameter to start both frontend and backend servers. Monitor output using the BashOutput tool
- If `pnpm dev` fails due to requiring interactive input, ask the user to run `pnpm convex dev --once` first in a separate terminal
- Import alias: `@/` maps to `src/` directory
- Tailwind CSS 4, daisyUI 5: All config in `src/index.css` via CSS syntax, NOT tailwind.config.js
- Typography: Uses `@tailwindcss/typography` with `prose prose-invert` at root level, use `not-prose` to escape (e.g., for buttons/tables)
- Environment variables: Client vars need `VITE_` prefix, Convex vars set in dashboard
- Package manager: Always use `pnpm` and `pnpx`, NOT `npm` or `npx`
- See @README.md for project-specific information

## Git Workflow

- Commit small pieces of work when possible
- If the user wants to roll back, use git to find the relevant commit to reset to

### Example good commit messages

- `pnpm install @convex-dev/aggregate`
- `rename: is_malicious -> maliciousness_score` (this commit might be part of a bigger refactor)
- `users: add getCurrentUserOrNull, getCurrentUserOrCrash`
- `schema: users: add bio`
- `frontend: calendar: add month view`

## Testing & Validation

- Before pushing:
  - Check background process output for Convex backend errors.
  - Run `pnpm lint` and `pnpm test:e2e`
  - Review with `git diff origin/main` (or whatever branch makes sense)
- Manual testing: Test UI with Playwright MCP (`mcp__playwright__browser_*`) before writing e2e tests
  - The playwright mcp server is unreliable, if it doesn't work ask the user to test manually
- Test account: `claude+clerk_test@example.com`, code `424242`. Use slowly: true / pressSequentially to trigger auto distribution
- Responsive testing: Use `mcp__playwright__browser_resize` to test mobile (375x667), tablet (768x1024), desktop (1200x800)
- Debug with `mcp__playwright__browser_console_messages` to view browser console output
- Add e2e tests only when explicitly requested by user - not proactively
- Convex in tests: Use `ConvexTestingHelper` for queries/mutations
- Test cleanup: Use `testingMutation` from `convex/testingFunctions.ts` for cleanup functions - prevents accidental production use
- If you run into an issue you don't know how to fix, look for relevant documentation or a reference implementation

## Convex

- `_creationTime` and `_id` are automatically added to all documents.
- Adding required fields breaks existing data - if early in development, ask the user to clear the database. Otherwise, plan migration.
- Always throw `ConvexError` for user-facing errors; frontend reads `error.data`, not `error.message`
- Queries have 16MB/10s limits - always use indexes, never full table scans
- Paginated queries: use `.paginate(paginationOpts)` with `paginationOptsValidator`
- Scheduled tasks: `ctx.scheduler.runAfter(delay, internal.module.function, args)` or `ctx.scheduler.runAt(timestamp, ...)`
- Unique fields: enforce in mutation logic, indexes don't guarantee uniqueness
- Soft delete: add `deletedAt: v.optional(v.number())` field instead of `.delete()`
- System tables: access `_scheduled_functions` and `_storage` with `ctx.db.system.get` and `ctx.db.system.query`
- Default query order is ascending by `_creationTime`
- Transactions are per-mutation - can't span multiple mutations. Calling multiple queries/mutation in a single action may introduce race conditions.
- Hot reload issues: Restart if schema changes don't apply or types are stuck
- Use `import { Doc, Id } from "./_generated/dataModel";` and `v.id("table")` for type safety.
- Add `"use node";` to the top of files containing actions that use Node.js built-in modules (can't contain queries and mutations)
- `"use node";` is NOT needed for fetch, only use it for other Node.js built-ins
- Convex + Clerk: Always use Convex's auth hooks (`useConvexAuth`) and components (`<Authenticated>`, `<Unauthenticated>`, `<AuthLoading>`) instead of Clerk's hooks/components. This ensures auth tokens are properly validated by the Convex backend.
- Import data with `pnpm convex import --table tableName file.json`

### Function guidelines

- Import `query`, `internalQuery`, `mutation`, `internalMutation`, `action`, `internalAction` from `./_generated/server` and call to register functions.
- Use `ctx.runQuery`, `ctx.runMutation`, `ctx.runAction` to call functions from other functions. e.g.: `import { api, internal } from "./_generated/api";` and then `ctx.runQuery(internal.module.function, { arg })`.
- If calling functions causes unexpected type errors, try adding a type annotation (helps circularity): `const result: string = await ctx.runQuery(api.module.function, { arg });`
- Actions can't directly access DB - use `ctx.runQuery` / `ctx.runMutation`

### Validator guidelines

- Always use an args validator for functions
- `v.bigint()` is deprecated for representing signed 64-bit integers. Use `v.int64()` instead.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Crazytieguy/fullstack-vibe-coding-template](https://github.com/Crazytieguy/fullstack-vibe-coding-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
