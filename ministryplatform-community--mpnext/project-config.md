---
trigger: always_on
description: This guide provides essential information for AI assistants (like Claude) working on the MPNext project.
---

# CLAUDE.md - MPNext Development Guide

This guide provides essential information for AI assistants (like Claude) working on the MPNext project.

## Ministry Platform Data Safety — MANDATORY

**NEVER delete, update, or create records in Ministry Platform without explicit user confirmation first.** No exceptions. No agents. No scripts. No "cleanup" operations. No "it's just one record." No "it's obviously safe."

Ministry Platform is a shared production database containing real church member data — contacts, communications, subscriptions, donations, groups, events. Unauthorized writes can affect thousands of people.

**Before ANY write operation** (`DELETE`, `UPDATE`, `INSERT`, or any API call that modifies data — including `deleteTableRecords`, `updateTableRecords`, `createTableRecords`, stored procedures that mutate state, or any HTTP `POST`/`PUT`/`DELETE` to the MP API):

1. **Stop.** Do not execute the operation.
2. **Show the user** exactly what will be affected: the table name, the record IDs, the fields that will change, and the old → new values where applicable.
3. **Wait for the user to explicitly say yes** before proceeding.
4. If the user says no, do not retry or suggest alternatives unless asked.

**Read-only operations are always fine** — `SELECT` queries, `GET` requests, `getTableRecords` calls. Only writes require confirmation.

**This rule applies to:**
- The main agent
- All subagents and background agents
- All general-purpose, Explore, and Plan agents
- All one-off scripts (e.g., `npx tsx` scripts)
- All automated cleanup, migration, or fix operations
- All hooks and scheduled tasks

**There are zero exceptions.** If you think "this is obviously safe and I don't need to ask," you are wrong. Ask anyway.

## Commands

- **Dev**: `npm run dev` (Next.js dev server)
- **Build**: `npm run build` (production build with Turbopack, runs type checking)
- **Lint**: `npm run lint` (ESLint CLI — `next lint` was removed in Next.js 16)
- **Generate MP Types**: `npm run mp:generate:models` (generates TypeScript types + Zod schemas from Ministry Platform API, cleans output directory first)
- **Tests**: `npm test` (Vitest in watch mode), `npm run test:run` (single run), `npm run test:coverage` (with coverage)
- **Setup**: `npm run setup` (interactive project setup wizard), `npm run setup:check` (validate setup without changes)

### Type Generation Notes

- Generated types automatically quote field names with special characters (e.g., `"Allow_Check-in"`)
- The `mp:generate:models` script uses `--clean` flag to remove old files before regenerating
- Manual generation with options: `tsx src/lib/providers/ministry-platform/scripts/generate-types.ts --help`

## Architecture

- **Framework**: Next.js 16 (App Router, Turbopack) with React 19, TypeScript strict mode
- **Ministry Platform Integration**: Custom provider at `src/lib/providers/ministry-platform/` with REST API client, auth, and type-safe models
- **Auth**: Better Auth with Ministry Platform OAuth via genericOAuth plugin — see **[Auth Reference](.claude/references/auth.md)** for full details
  - **Key files**: `src/lib/auth.ts` (server config), `src/lib/auth-client.ts` (client), `src/proxy.ts` (route protection)
  - **Critical**: `session.user.id` is Better Auth's internal ID, NOT the MP User_GUID. Use `session.user.userGuid` for all MP API lookups.
  - **Stateless Sessions**: JWT cookie cache, no database; `customSession` does name splitting only (no API calls)
  - **Required Environment Variables**: `MINISTRY_PLATFORM_BASE_URL`, `BETTER_AUTH_URL` (or `NEXTAUTH_URL` fallback), `BETTER_AUTH_SECRET` (or `NEXTAUTH_SECRET` fallback)
- **Services Layer**: Singleton service classes in `src/services/` wrap MPHelper for domain logic (ContactService, ContactLogService, ToolService, UserService)
- **Contexts**: React context providers in `src/contexts/` (UserProvider) composed in `src/app/providers.tsx`; `useAppSession()` wraps Better Auth's `authClient.useSession()`
- **UI**: Radix UI primitives + shadcn/ui components in `src/components/ui/`, Tailwind CSS v4
- **Validation**: Zod v4 (`zod@^4.3`) — note: different API from Zod v3 (e.g., `z.object()` vs `z.interface()`)
- **Path Alias**: `@/*` maps to `src/*`

## Next.js 16 Notes

- **Proxy (formerly Middleware)**: Route protection lives in `src/proxy.ts` with an exported `proxy()` function (not `middleware.ts`/`middleware()`)
- **Turbopack**: Default bundler for both `dev` and `build` — no `--turbopack` flag needed
- **ESLint**: Uses `eslint .` directly (not `next lint`); config is native flat config in `eslint.config.mjs`
- **Async Dynamic APIs**: `params`, `searchParams`, `cookies()`, `headers()` must always be awaited — synchronous access is removed
- **Dev output**: `next dev` outputs to `.next/dev` (not `.next`)

## Code Style

- **Imports**: Use `@/` alias for all internal imports
- **Components**: React Server Components by default, "use client" only when needed for interactivity
- **Types**: TypeScript interfaces exported from models, Zod schemas for validation
- **Naming**:
  - PascalCase for components/types
  - camelCase for functions/variables
  - kebab-case for all component files and folders
  - snake_case for Ministry Platform API fields

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MinistryPlatform-Community/MPNext](https://github.com/MinistryPlatform-Community/MPNext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
