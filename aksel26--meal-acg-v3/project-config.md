---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md (English)

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Meal ACG v3 - Enterprise meal expense management application

## Development Commands

```bash
# Install dependencies
pnpm install

# Dev servers
pnpm dev          # Start all dev servers (user:3000, admin:3001)
pnpm dev:user     # Start only user app (port 3000)
pnpm dev:admin    # Start only admin app (port 3001)

# Build
pnpm build        # Build entire monorepo
pnpm build:user   # Build only user app
pnpm build:admin  # Build only admin app

# Production
pnpm start:user   # Start production user app (after build)
pnpm start:admin  # Start production admin app (after build)

# Code quality
pnpm lint         # ESLint (max warnings = 0)
pnpm check-types  # TypeScript type checking
pnpm format       # Prettier formatting
```

## Architecture

**Monorepo** using Turborepo + pnpm workspaces:

```
apps/
  user/              # User-facing Next.js 15 app (port 3000)
  admin/             # Admin dashboard Next.js 15 app (port 3001)

packages/
  ui/                # Shared Radix UI components (@repo/ui)
  utils/             # Shared utilities - dayjs, KST date functions (@repo/utils)
  eslint-config/     # Shared ESLint config
  typescript-config/ # Shared TypeScript config
  tailwind-config/   # Shared Tailwind config
```

## Tech Stack

- **Framework:** Next.js 15 (App Router, Turbopack), React 19
- **Language:** TypeScript 5 (strict mode)
- **Styling:** Tailwind CSS 4, Motion (animations), Radix UI
- **State:** Zustand (client), TanStack React Query (server)
- **Backend:**
  - user app: Supabase, Google APIs (Sheets, Calendar), Gemini AI (receipt scanning)
  - admin app: Supabase (SSR client + RLS, service client for admin ops)
- **Build:** Turborepo, pnpm

## Key Patterns

### Query Keys (Factory Pattern)
Centralized in each app's `lib/query-keys.ts`:
```typescript
// User app
queryKeys.meals.byUserAndMonth(userName, month, year)
queryKeys.points.activity.byPeriod(memberId, period)

// Admin app
queryKeys.stats.monthly(year, month)
queryKeys.memberStatuses.all
queryKeys.budgetAllocations.byPeriod(period)
queryKeys.usageRecords.byPeriod(period)
```

### Query Invalidation Pattern
**Critical:** When mutating data, invalidate related queries to refresh UI. Common patterns:

```typescript
// Member mutations (add/delete/edit)
queryClient.invalidateQueries({ queryKey: ["dashboard"] });
queryClient.invalidateQueries({ queryKey: queryKeys.stats.all });
queryClient.invalidateQueries({ queryKey: queryKeys.members.all });
queryClient.invalidateQueries({ queryKey: queryKeys.memberStatuses.all });

// Member status changes (퇴사 등)
queryClient.invalidateQueries({ queryKey: ["dashboard"] });
queryClient.invalidateQueries({ queryKey: queryKeys.stats.all });
queryClient.invalidateQueries({ queryKey: queryKeys.memberStatuses.all });

// Organization mutations (team/division assignment)
queryClient.invalidateQueries({ queryKey: queryKeys.memberStatuses.all });
// ↑ member_current_status is a VIEW that joins teams/divisions

// Usage records mutations
queryClient.invalidateQueries({ queryKey: queryKeys.usageRecords.all });
queryClient.invalidateQueries({ queryKey: queryKeys.budgetSummary.all });
```

**Key insight:** `member_current_status` is a Supabase **VIEW** (not table) that joins `members + member_statuses + teams + divisions`. Always invalidate `memberStatuses.all` when updating any of these entities.

**Note:** This VIEW does not include `role` (admin/user). To check member roles, query `members` table directly or cross-reference via `allMembers` query.

### Data Fetching Hooks
Custom hooks in `hooks/` wrapping React Query with proper typing:
```typescript
// Pattern: use-{resource}.ts or use-{resource}-{action}.ts
useMealData(userName, month, year)       // query hook
useMealSubmit()                          // mutation hook
useDeleteUsageRecords()                  // bulk mutation hook
useAllUsageRecords(filters)              // infinite query hook
```

**Common patterns:**
- **Single resource:** `use-{resource}.ts` (e.g., `use-meal-data.ts`)
- **Mutation:** `use-{resource}-{action}.ts` (e.g., `use-meal-submit.ts`)
- **Bulk operations:** `useDelete{Resources}` (plural, accepts `ids: string[]`)
- **Infinite scroll:** Use `useInfiniteQuery` with `limit`/`offset` params

### Zustand Stores
Located in each app's `stores/`:
```typescript
// User app
useUserStore()        // Auth state with localStorage persistence
useMealDrawerStore()  // UI state

// Admin app
useAuthStore()        // Admin session state
```

### Shared Packages
- `@repo/ui`: Radix-based components (Button, Dialog, Drawer, etc.)
- `@repo/utils`: KST timezone date utilities (formatDate, getToday, etc.)

**Import convention:** Always use `@repo/ui/src/{component}` (e.g., `import { toast } from "@repo/ui/src/sonner"`), not direct package imports like `from "sonner"`.

### Tailwind CSS 4
Uses CSS-first config (`@import "tailwindcss"` + `@import "@repo/tailwind-config"`). No `tailwind.config.ts` — custom theme in `packages/tailwind-config/shared-styles.css` using `@theme` directive.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aksel26/meal-acg-v3](https://github.com/aksel26/meal-acg-v3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
