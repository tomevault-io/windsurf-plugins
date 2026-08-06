---
trigger: always_on
description: Internal web tool for a Viennese fitness supplement store (MuscleZone).
---

# MuscleZone Pricetag Generator — AI Handoff Context

## What This Is

Internal web tool for a Viennese fitness supplement store (MuscleZone).
Purpose: generate A4 price-tag sheets (print/PDF) sourced from the helloCash POS system.
Live at **https://pricetag.musclezone.at** (Vercel, auto-deploys from `main`).

Replaces a PHP/HTML single-page app. Same UX, modern stack.

---

## Stack

| Layer | Tech |
|---|---|
| Framework | Next.js 16 App Router + React 19 + TypeScript |
| Styling | Tailwind CSS v4 |
| Database | Supabase Postgres |
| Auth | Supabase Auth — **currently DISABLED** (see proxy.ts) |
| Deployment | Vercel (branch `main` → auto-deploy) |
| POS source | helloCash REST API v1 (read-only, server-side only) |

---

## Breaking: This Is NOT Standard Next.js

<!-- BEGIN:nextjs-agent-rules -->
Next.js 16 has breaking changes from training data:
- **Middleware file is `src/proxy.ts`**, NOT `middleware.ts`
- Read `node_modules/next/dist/docs/` before changing routing/middleware
- App Router is used throughout (`src/app/`)
<!-- END:nextjs-agent-rules -->

---

## Auth Status: DISABLED

`src/proxy.ts` passes all requests through without auth checks:
```typescript
export function proxy() { return NextResponse.next(); }
export const config = { matcher: ['/((?!_next/static|_next/image|favicon.ico).*)'] };
```
This is intentional — the store is internal and the client prefers no login.
To re-enable: restore Supabase session check in proxy.ts.

---

## Environment Variables (set in Vercel, NOT in .env.local)

| Variable | Visibility | Purpose |
|---|---|---|
| `NEXT_PUBLIC_SUPABASE_URL` | Client + Server | Supabase project URL |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Client + Server | Anon/publishable key (RLS) |
| `SUPABASE_SECRET_KEY` | **Server only** | Service-role key (admin client, sync) |
| `HELLOCASH_API_TOKEN` | **Server only** | helloCash Bearer token (read-only) |
| `HELLOCASH_API_BASE_URL` | Server | Default: `https://api.hellocash.business/api/v1` |

**Important:** Code accepts both `NEXT_PUBLIC_SUPABASE_ANON_KEY` and
`NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` (fallback) because the var was renamed.
`NEXT_PUBLIC_*` vars are embedded at build time — a rebuild is needed after changing them.

---

## Database Schema

Migrations live in `supabase/migrations/` (additive, non-destructive).
Migrations 0001, 0002, 0003 are **already applied** in the live Supabase project.

| Table / View | Role |
|---|---|
| `products` | Mirror of helloCash articles (managed by sync) |
| `product_overrides` | Local edits — NEVER overwritten by sync |
| `categories` | Category lookup (name → id) |
| `sync_cache` | SHA-256 hash per external article for incremental sync |
| `sync_runs` | Sync log: created / updated / unchanged / skipped counts |
| `label_settings` | Label layout, fonts, dimensions |
| `label_selection_sets` / `label_selection_items` | Saved product selections |
| `effective_products` (view) | `coalesce(override, source)` — what the UI reads |

`effective_products` view merge logic: for each field (name, price, category, EAN, icon),
the override wins over the source value.

### RLS Policies (migration 0003)

Since auth is disabled, anon users must be able to read/write:
- `product_overrides`: anon full access
- `sync_cache`: anon full access
- `label_selection_sets` / `label_selection_items`: anon full access
- `sync_runs`: anon read-only; service_role write-only

**Admin client** (`getSupabaseAdmin()` using `SUPABASE_SECRET_KEY`) bypasses RLS.
Use it for: sync writes, reading sync_runs, saving overrides, label selections, settings.
Do NOT use `createClient()` (anon) for mutations — it goes through RLS.

---

## Supabase Client Usage Rules

```typescript
// Server-side admin (bypasses RLS) — for sync, mutations, reads that need all rows
import { getSupabaseAdmin } from '@/lib/supabase/admin';
const supabase = getSupabaseAdmin();

// Server-side anon (subject to RLS) — for public reads only
import { createClient } from '@/lib/supabase/server';
const supabase = createClient();
```

`getSupabaseAdmin()` is **lazy** — throws at call time, not at build time, when `SUPABASE_SECRET_KEY` is missing.
`createClient()` must be called **inside try/catch** in server components (Next.js 16 lint rule).
JSX must **not** be rendered inside try/catch blocks (Next.js 16 lint rule).

### Supabase Row Limit

Supabase returns max 1000 rows per query by default. Use `.range()` pagination in a loop:
```typescript
let from = 0;
const CHUNK = 1000;
while (true) {
  const { data } = await supabase.from('effective_products').select('*').range(from, from + CHUNK - 1);
  all.push(...data ?? []);
  if ((data?.length ?? 0) < CHUNK) break;
  from += CHUNK;
}
```
See `src/lib/products/queries.ts` for the full implementation.

---

## helloCash API

Base URL: `https://api.hellocash.business/api/v1`
Auth: `Authorization: Bearer <HELLOCASH_API_TOKEN>`
Mode: **read-only** — never write back to helloCash.

### Pagination (confirmed from live API debug)

First call: `GET /articles` (no params)
Response shape:
```json
{
  "articles": [...],
  "count": "2031",   // STRING, not number — total article count
  "limit": 250,      // page size
  "offset": 1        // 1-based page number (NOT item offset)
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GAINS1210/generator](https://github.com/GAINS1210/generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
