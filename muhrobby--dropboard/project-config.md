---
trigger: always_on
description: Full-stack Next.js 16 (App Router) asset management platform. TypeScript strict mode, Tailwind CSS v4, Drizzle ORM + PostgreSQL, TanStack Query, Zustand, better-auth, shadcn/ui.
---

# AGENTS.md — Dropboard Coding Guidelines

## Project Overview

Full-stack Next.js 16 (App Router) asset management platform. TypeScript strict mode, Tailwind CSS v4, Drizzle ORM + PostgreSQL, TanStack Query, Zustand, better-auth, shadcn/ui.

---

## Commands

```bash
# Dev server (port 3004, not 3000)
pnpm dev

# Production build
pnpm build

# Lint (ESLint 9 flat config)
pnpm lint

# Type-check (no emit)
npx tsc --noEmit

# Run all tests
pnpm test                          # vitest run (CI mode, no watch)
pnpm test:watch                    # vitest (watch mode)

# Run a single test file
pnpm test -- __tests__/item-service.test.ts

# Run tests matching a name pattern
pnpm test -- --reporter=verbose -t "should create item"

# Database
pnpm db:push        # sync schema changes to DB (after schema edits)
pnpm db:generate    # generate migration files
pnpm db:migrate     # apply migrations
pnpm db:studio      # open Drizzle Studio
pnpm db:seed        # seed the database
```

### Package Manager

Use **pnpm** exclusively (`pnpm-lock.yaml` present).

```bash
pnpm add <pkg>       # runtime dependency
pnpm add -D <pkg>    # dev dependency
```

---

## Tech Stack

| Layer | Library |
|---|---|
| Framework | Next.js 16, App Router, React 19 |
| Language | TypeScript 5, strict mode |
| Styling | Tailwind CSS v4, `@import "tailwindcss"` in globals.css |
| UI components | shadcn/ui (in `components/ui/`) |
| Fonts | `Inter` (sans) + `JetBrains_Mono` (mono) via `next/font/google` |
| Server state | TanStack Query v5 |
| Client state | Zustand v5 |
| ORM | Drizzle ORM + `postgres` driver |
| Database | PostgreSQL, ULID primary keys |
| Auth | better-auth, session cookies |
| Validation | Zod v4 — always import from `"zod/v4"` |
| Testing | Vitest 4 (integration, real DB, no mocks) |

---

## Project Structure

```
app/                  # Next.js App Router
  (auth)/             # Login, register, password reset
  api/v1/             # All REST API routes
  dashboard/          # Main app pages
  actions/            # Next.js server actions
components/
  drops/              # Drop-specific components
  shared/             # Generic reusable components
  todos/              # Todo/task components
  ui/                 # shadcn/ui primitives (do not edit)
  layout/             # Sidebar, topbar, nav
db/
  schema/             # Drizzle table definitions + index.ts barrel
  index.ts            # db instance (drizzle + postgres)
hooks/                # TanStack Query hooks (use-items.ts pattern)
lib/
  api-helpers.ts      # Typed NextResponse helpers
  errors.ts           # AppError hierarchy
  validations/        # Zod schemas + inferred types
  constants.ts        # App-wide constants + RBAC permission maps
  tier-guard.ts       # Subscription/quota checks
  file-storage.ts     # Disk storage + signed URLs
middleware/           # auth-guard.ts, workspace-guard.ts, rbac.ts
services/             # Business logic (item-service.ts pattern)
stores/               # Zustand stores (workspace-store, ui-store)
types/
  api.ts              # API response types (ApiResponse<T>, ItemResponse, etc.)
  index.ts            # Domain enums (ItemType, MemberRole, etc.)
__tests__/            # All Vitest tests live here
```

---

## Testing

Tests use **Vitest** against a **real PostgreSQL database** — no mocks, sequential execution.

- All test files go in `__tests__/` (the vitest config only picks up `__tests__/**/*.test.ts`)
- Setup file: `__tests__/setup.ts` loads `.env.local.test`
- Test DB: configure `DATABASE_URL` in `.env.local.test`
- Baseline: **103 passed / 1 skipped** — all must pass before merging
- `fileParallelism: false` — tests run sequentially, do not add parallelism

```bash
# Single file
pnpm test -- __tests__/item-service.test.ts

# Single test by name
pnpm test -- -t "createItem"
```

---

## Code Style

### TypeScript

- Strict mode is on — no `any`, no non-null assertions (`!`) without justification
- Prefer `type` over `interface` for object shapes
- Use inferred types from Zod schemas: `type CreateDropInput = z.infer<typeof createDropSchema>`
- Use `React.ReactNode` for children props
- Always annotate async function return types explicitly

### Imports

- Use `@/*` alias (maps to repo root): `import { db } from "@/db"`
- Import order: React/Next → third-party → local `@/` imports
- Use `import type` for type-only imports

```typescript
import { NextRequest, NextResponse } from "next/server";
import { z } from "zod/v4";
import { db } from "@/db";
import type { ItemResponse } from "@/types/api";
```

### Zod — always use `"zod/v4"`

```typescript
import { z } from "zod/v4";   // correct
import { z } from "zod";       // wrong — will break at runtime
```

### Naming

| Thing | Convention |
|---|---|
| Components | PascalCase (`DropCard.tsx`) |
| Utility files | kebab-case (`file-storage.ts`) |
| Variables / functions | camelCase |
| Constants | UPPER_SNAKE_CASE |
| DB column names | snake_case (Drizzle maps to camelCase via `.$defaultFn`) |

### Components

- `"use client"` at top of any component using hooks, event handlers, or browser APIs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/muhrobby) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
