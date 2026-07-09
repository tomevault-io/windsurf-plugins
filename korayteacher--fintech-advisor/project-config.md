---
trigger: always_on
description: - Foliofox is a Next.js App Router application for portfolio intelligence and net worth tracking
---

# AGENTS Guidelines for This Repository

## Project Context

- Foliofox is a Next.js App Router application for portfolio intelligence and net worth tracking
- Stack: TypeScript, Supabase, Shadcn UI, Tailwind CSS
- Deployment: Vercel
- Data Strategy: Daily cron refreshes for market data/FX and manual updates (no WebSocket)
- Canonical data model: positions (assets and future liabilities), portfolio_records (buy|sell|update), position_snapshots

## Development Principles

- Prefer server-first architecture (RSC, minimal client-only code).
- Keep data access in `server/` with `"use server"` and the correct Supabase client.
- Favor explicit, typed, functional code over clever abstractions.
- Optimize for cache/batching; avoid N+1 database calls.

## Planning & Execution Guardrails

- For phased plans, stop after each phase and wait for explicit user approval before continuing.
- Do not run Supabase CLI commands, Next CLI commands, or any command against any database or production system.
- Allowed verification commands are limited to lint, type, format-check, and test checks.
- Do not create Supabase migration files. Ask the user to create an empty migration file first, then edit that file only after it exists.
- Once the migration file is ready, the user will apply the migration against the local database and they will regenerate the types.

## Architecture

- Favor React Server Components (RSC) over Client Components
- Keep URL state minimal; prefer lightweight search param helpers when needed
- Minimize 'use client' directives - only for Web API/browser features
- Split complex features into modular components
- Keep server actions/functions in `server/` with `"use server"`
- Supabase: use user-scoped client from `supabase/server.ts`; reserve service-role client from `supabase/service.ts` for cron/admin only (bypasses RLS)

## TypeScript & Code Style

- Use TypeScript for all code with strict type checking
- Prefer interfaces for component props and public APIs
- Use types for utility, mapped, or database types (e.g., generated types, unions)
- Use descriptive names with auxiliary verbs (isLoading, hasError)
- Follow functional programming patterns
- Avoid unnecessary class usage and type casting
- Prefer maps or const objects over TypeScript enums
- Avoid wrapping async functions in explicit `new Promise`
- Prefer explicit named boolean flags (e.g., `includeSnapshots: true`) over positional booleans
- Centralize runtime enums in `types/enums.ts` (DB-backed constants), and import where needed
- Prefer explicit and self-descriptive code; avoid abbreviations or initials
- Add concise comments for non-obvious business logic and multi-step calculations; avoid comments that only restate code.
- When a function already uses numbered flow comments (`1.`, `2.`, `3.`), preserve that structure when editing it.

## File Structure

```
components/
├── ui/                # Shadcn components (do not modify)
├── ai-elements/       # AI SDK Elements components (do not modify)
├── features/          # Cross-route feature components
├── dashboard/         # Dashboard route-group components
├── homepage/          # Marketing homepage components
└── public-portfolio/  # Public portfolio views
app/
├── (dashboard)/       # Authenticated dashboard routes
├── (public)/          # Public/marketing routes
├── api/
├── auth/
└── maintenance/
lib/
types/
hooks/
server/
supabase/              # Clients, migrations, seed

```

## Import Order

```typescript
// 1. External Dependencies
import { type FC } from "react";
import { createClient } from "@supabase/supabase-js";

// 2. UI Components
import { Button } from "@/components/ui/button";

// 3. Custom Components
import { LoginForm } from "@/components/features/auth/login-form";

// 4. Internal Modules
import { useCurrencies } from "@/hooks/use-currencies";

// 5. Local Files and Types
import type { Position } from "@/types/global.types";
```

## Component Structure

```typescript
// Types and interfaces first
interface Props {
  // ...
}

// Component with explicit return type
export function ComponentName({ prop1, prop2 }: Props) {
  // 1. Hooks
  // 2. Derived state
  // 3. Event handlers
  // 4. Render
}
```

- Use named exports for components, except for Next.js route files, which require default exports.

## Currency & Data Handling

- We mostly use ISO currency codes (USD, EUR) - unless explicitly requested
- Handle all monetary values as numbers, format only for display
- Use Zod for data validation; for AI tool schemas use `.nullable()` for optional fields (https://ai-sdk.dev/docs/ai-sdk-core/prompt-engineering#optional-parameters)
- Market data: use the aggregator `server/market-data/fetch.fetchMarketData()` (handlers registry). Default to `upsert: true` so DB cache is populated.
- Valuation: `fetchPositions({ asOfDateKey })` uses snapshot quantity and market price at as-of date for unit value (fallback to snapshot value for custom positions).
- Profit/Loss: compute basis from latest relevant snapshot; do not derive from current market price.

## Date & Timezone Handling


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KORAYTEACHER/fintech-advisor](https://github.com/KORAYTEACHER/fintech-advisor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-09 -->
