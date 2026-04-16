---
trigger: always_on
description: This repository is the **app** (served on `app.recordcadence.com`). Marketing lives in a different repo.
---

# AGENTS.md — Cadence App Engineering Rules

This repository is the **app** (served on `app.recordcadence.com`). Marketing lives in a different repo.
Your job is to ship a clean MVP with long-term maintainability, minimal rework, and low-bug velocity.

If you violate these rules, stop and fix before proceeding.

---

## 0) Non-negotiables

- **Never ask for or print secrets** (keys, tokens, cookies, env values). You may reference env var _names only_.
- **Minimal, auditable diffs.** No drive-by refactors. One feature per PR.
- **Default route design:** app base is `/` and the authenticated landing page is **`/log`**.
  - Do **not** create or assume an `/app` base path.
  - Route groups like `(app)` do **not** create URL segments.

---

## 1) Tech stack assumptions

- Next.js App Router + React + TypeScript
- Tailwind + shadcn/ui + tokenized CSS variables
- pnpm
- Supabase (when enabled): `@supabase/ssr` + `@supabase/supabase-js`
- Forms: React Hook Form + Zod + `@hookform/resolvers` (follow form discipline)

---

## 2) Architecture principles (Staff-level defaults)

### Server-first

- **Prefer Server Components** and server data fetching.
- Add `'use client'` only when required (forms, camera, local stateful UI, client-only APIs).
- Avoid client-side data fetching unless strictly necessary for interactivity.

### No redirect loops / no effect-based navigation

- **Do not** use `useEffect` to implement auth redirects or route guarding.
- Guard on the server (`redirect()` in Server Components/layouts) and/or **middleware**.
- Centralize post-auth destination in a single place (e.g., `getPostAuthDestination()` returns `/log`).

### Clean architecture: UI-agnostic domain layer (required)

**Rule: All business logic for students, logging, and auth helpers must be UI-agnostic and live outside React components.**

Separation of concerns:

**UI Layer (React / Next pages/components)**

- Only: layout, visual state, form wiring, calling domain functions, rendering.
- Must not: contain business rules, data validation rules, or Supabase queries (except in narrowly approved “bridge” helpers).

**Domain Layer (UI-agnostic) — required location**

- `src/lib/domain/*`
- Contains: types, validation schemas, business rules, pure functions.
- ✅ Must be importable from future React Native without changes.
- ❌ Must not import from `react`, `next/*`, `@/components/*`, or browser globals.

**Data Access Layer (UI-agnostic, runtime-aware)**

- `src/lib/data/*`
- Contains: repository functions (CRUD) and query builders.
- Should accept an injected client or use a thin adapter.
- ❌ No direct component imports.
- ✅ May depend on Supabase client types and small adapters.

**Platform/Framework Adapters (thin wrappers only)**

- `src/lib/platform/*` or `src/lib/supabase/*`
- Contains: Next/Supabase SSR cookie wiring, middleware helpers, request-context creation.
- Purpose: isolate Next.js and Supabase SSR specifics so domain/data remain portable.

Folder conventions (hard requirement)

- `src/lib/domain/students.ts` → student rules + types + validation
- `src/lib/domain/activities.ts` → activity/log rules + types + validation
- `src/lib/domain/auth.ts` → auth-related helpers (redirect target selection, safe path, etc.)
- `src/lib/data/students.repo.ts` → persistence functions for students
- `src/lib/data/activities.repo.ts` → persistence functions for logs/activities
- `src/lib/supabase/{client,server}.ts` → only client factories / SSR cookie wiring

Import rules (enforced by convention)

Domain modules MUST NOT import:

- `react`, `next/*`, `@/components/*`, `window`, `document`
- CSS, Tailwind, shadcn, UI code

Data modules MUST NOT import:

- `@/components/*`
- `next/navigation` (except in explicit server action files if you choose that pattern)

UI MUST NOT:

- write raw Supabase queries
- embed business rules (e.g., “durationMinutes must be 1–1440”, “student name rules”, “log normalization”)

Patterns we prefer (to avoid useEffect loops)

- Prefer Server Components for initial reads.
- Prefer Server Actions or Route Handlers for writes.
- Client components only handle form input and call a server action / route.

React Native readiness checkpoint

Every feature PR must answer:

- “Can I import this into a React Native app without pulling Next/React dependencies?”
- “Are rules validated in domain, not duplicated in UI?”
- “Is Supabase access behind a repo function?”

---

## 3) React rules (avoid loops & footguns)

- Avoid `useEffect` by default.
- Prefer **derived values** over state.
- Never set state in an effect that depends on that same state.
- No “fetch in effect” for App Router pages.
- Avoid `window.location.*` unless you have a clear reason; prefer Next navigation APIs where appropriate.

---

## 4) Supabase auth rules (when enabled)

### Env vars (names only)

Standardize on:

- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`

Never introduce alternate names (e.g., PUBLISHABLE_KEY).

### SSR client construction

- Use **only** `@supabase/ssr`.
- Cookies interface must be **`getAll` / `setAll`** only.
- Auth callback must:
  1. exchange code for session
  2. write cookies on the response
  3. redirect to a **safe internal path** (default `/log`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nicholasmackey) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
