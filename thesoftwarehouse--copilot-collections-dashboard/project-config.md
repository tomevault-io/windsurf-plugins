---
trigger: always_on
description: > **This file is the project constitution.** Every rule below is a hard constraint — not a suggestion, not a guideline. Copilot must follow these rules in all interactions with this repository without exception. When in doubt, these instructions take precedence over general knowledge or external best practices.
---

> **This file is the project constitution.** Every rule below is a hard constraint — not a suggestion, not a guideline. Copilot must follow these rules in all interactions with this repository without exception. When in doubt, these instructions take precedence over general knowledge or external best practices.

# Architecture

Next.js 16 App Router application — a GitHub Copilot usage analytics dashboard with PostgreSQL backend. Standalone output mode for Docker deployment. No monorepo — single application.

Route group `(app)` wraps all authenticated pages with config + session checks.

```
src/
  app/           → Next.js App Router pages and API routes
    (app)/       → Route group for authenticated pages (layout checks config + session)
    api/         → REST API routes
  components/    → UI components organized by domain folder (usage/, dashboard/, teams/, shared/)
  entities/      → TypeORM EntitySchema definitions + TypeScript interfaces
  lib/           → Shared business logic and helpers
    hooks/       → Client-side React hooks
    validations/ → Zod schemas for request validation
    __tests__/   → Unit tests for lib modules
  test/          → Test infrastructure (setup, db-helpers)
migrations/      → TypeORM migration files (timestamped)
e2e/             → Playwright E2E tests
  helpers/       → E2E helper utilities (auth, db)
specifications/  → Feature specs, task plans, story plans
```

# Technology Stack

- TypeScript 5 (strict mode, path alias `@/*` → `./src/*`)
- Next.js 16.1.6 (App Router, standalone output)
- React 19.2.3 with Server Components
- Tailwind CSS 4 via `@tailwindcss/postcss`
- TypeORM 0.3.28 with EntitySchema pattern (NOT decorators)
- PostgreSQL 16 via `pg` 8.19
- Zod 4.3.6 for request validation
- Recharts 3.7.0 for charts
- arctic 3.7.0 for Azure Entra ID OAuth
- bcryptjs 3.0.3 for password hashing
- node-cron 4.2.1 for scheduled jobs
- Vitest 4.0.18 for unit/integration tests
- Playwright 1.58.2 for E2E tests
- ESLint 9 with eslint-config-next

# Coding Conventions

Rules not enforced by linters:

**Path aliases only** — use `@/` (maps to `src/`) for all imports. Never use relative imports across directories.

**No barrel files** — import directly from each module. No `index.ts` re-exports.

**Entity definition style** — always `EntitySchema<Interface>`, never decorators. Each entity file exports both the TypeScript interface and the EntitySchema:

```ts
// Preferred
export interface Team { id: number; name: string; }
export const TeamEntity = new EntitySchema<Team>({ name: "Team", tableName: "team", columns: { ... } });

// Avoided — never use decorators
@Entity() class Team { @PrimaryGeneratedColumn() id: number; }
```

**Database access** — always through `getDb()` from `@/lib/db`. Never import or construct DataSource directly in route handlers.

**Zod schema naming** — `create{Entity}Schema`, `update{Entity}Schema` in `src/lib/validations/`. Shared field schemas in `shared.ts`. Extract types with `z.infer<typeof schema>`.

**Type guard pattern** — use `result is Type` return types for discrimination: `isAuthFailure()`, `isValidationError()`, `isUniqueViolation()`.

**Custom error classes** — extend `Error` with `this.name = "ClassName"` in constructor. `NotFoundError` in `@/lib/errors`.

**Migration format** — `{timestamp}-{PascalCaseDescription}.ts` in `migrations/`. Raw SQL in `up()`/`down()`. Production uses `synchronize: false`.

# Component Patterns

- All UI components in `src/components/` are client components — every file starts with `"use client"`.
- Components organized by domain folder: `usage/`, `dashboard/`, `teams/`, `shared/`.
- Default export pattern: `export default function ComponentName()`.
- Props defined as inline interfaces: `ComponentNameProps`.
- Data fetching: `useState` + `useEffect` with `cancelled` flag for cleanup — no SWR or React Query.
- Three-state rendering pattern: loading (`role="status"`) → error (`role="alert"`) → data.
- Tailwind utility classes inline — no CSS modules or styled-components.
- ARIA roles for accessibility: `role="status"` for loading, `role="alert"` for errors.
- Local sub-components (helper components used only in one file) are defined as functions in the same file, not exported.

# Error Handling

- API routes: single `try/catch` wrapping the entire handler body, `catch` delegates to `handleRouteError(error, "METHOD /api/path")`
- `handleRouteError` returns: `NotFoundError` → 404, unique violation → 409, generic → 500
- Error response format: `{ error: "message" }`, optionally `{ error, details }` for validation errors
- Never expose internal error details in production

# API Route Lifecycle

Every API route handler follows this exact sequence:

1. **Auth guard** — `const auth = await requireAuth(); if (isAuthFailure(auth)) return auth;`
2. **Validation** — `const parsed = await validateBody(request, schema); if (isValidationError(parsed)) return parsed;`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheSoftwareHouse/copilot-collections-dashboard](https://github.com/TheSoftwareHouse/copilot-collections-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
