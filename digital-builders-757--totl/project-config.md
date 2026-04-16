---
trigger: always_on
description: **BEFORE WRITING ANY CODE, YOU MUST:**
---

````markdown
# TOTL Agency – Cursor AI Assistant Rules

## 🎯 MANDATORY CONTEXT REFERENCE

**BEFORE WRITING ANY CODE, YOU MUST:**

1. **READ** `AGENT_ONBOARDING.md` (NEW AGENTS START HERE)
2. **READ** `TOTL_PROJECT_CONTEXT_PROMPT.md` completely
3. **READ** `docs/ARCHITECTURE_CONSTITUTION.md` (MANDATORY — system boundaries)
4. **CHECK** `supabase/migrations/` for current database schema
5. **VERIFY** RLS policies in the context file
6. **USE** generated types from `types/database.ts` / `@/types/supabase` (NO `any` types)
7. **FOLLOW** established project structure and naming conventions

If any of these files are missing from the context, proceed with best effort and flag the missing doc(s) in the plan (and any assumptions you’re making).

---

## 🧱 Architecture Constitution (MANDATORY)

Before implementing any feature, read:

- `docs/ARCHITECTURE_CONSTITUTION.md`

Non-negotiables:

- Middleware is security only; never create/update DB rows in middleware.
- Supabase Auth is identity; profiles is app identity; missing profile is a valid state during bootstrap.
- Mutations go through Server Actions / API routes only.
- Client components do not perform DB writes or direct privileged reads.
- RLS is the final authority; do not bypass with service role in client code.
- No `select('*')`; explicit columns.
- Never manually edit generated types.

If your change touches auth, middleware, redirects, profiles, onboarding, Stripe webhooks, or RLS:

1) Summarize the constitution in **5 bullets**.
2) Propose the change as a **minimal diff**.
3) List **risk points** and **tests** (redirect loops, bootstrap, RLS, webhook retries).

If modifying middleware/auth callback/profile bootstrap:

- Explain **why** the change is needed and **how it prevents redirect loops** (safe routes, signedOut handling, etc.).

If modifying Stripe webhooks:

- Must be **idempotent** (safe to retry), and must verify Stripe signatures.

## 🚨 RED ZONE FILES (NO BLIND REWRITES)

These files control **auth, routing, and schema truth**. They must **never** be auto-rewritten or heavily refactored in a single pass.

**Red Zone – Auth & Routing**

- `components/auth/auth-provider.tsx`
- `middleware.ts`
- `app/auth/callback/page.tsx`
- `app/talent/dashboard/page.tsx`
- `app/talent/dashboard/client.tsx`
- `app/client/dashboard/page.tsx`
- `app/client/dashboard/client.tsx`
- `app/(auth)/**` (login, signup, verification, choose-role, reset-password flows)
- `app/api/auth/*`
- `app/api/stripe/webhook/route.ts` (must be idempotent)
- `docs/SIGN_OUT_IMPROVEMENTS.md`
- `docs/contracts/AUTH_BOOTSTRAP_ONBOARDING_CONTRACT.md`
- `docs/TALENT_DASHBOARD_FLOW.md` (if present)

**Red Zone – Schema & Types**

- `database_schema_audit.md`
- `supabase/migrations/**`
- `types/database.ts`
- `types/supabase.ts` (or equivalent re-export)
- `docs/DATABASE_REPORT.md`

**Red Zone – Project Context**

- `TOTL_PROJECT_CONTEXT_PROMPT.md`
- `.cursorrules`

### Red Zone Editing Protocol (MANDATORY)

When modifying ANY Red Zone file:

1. **Show current file contents** (or at least the relevant section).
2. **Summarize the existing behavior** in plain language.
3. **Propose a small, explicit change** (minimal diff) – not a full rewrite.
4. **Apply only that change**, preserving existing working logic.
5. **Re-read** `docs/contracts/AUTH_BOOTSTRAP_ONBOARDING_CONTRACT.md`, `docs/SIGN_OUT_IMPROVEMENTS.md`, and `database_schema_audit.md` if the change touches:
   - auth state
   - middleware behavior
   - dashboards
   - database schema
6. **Never**:
   - Convert a Red Zone file into a “brand new pattern” all at once.
   - Remove working logic without explaining why.
   - Introduce `window.location.reload()` into auth/dashboards.

If a large refactor is truly needed, it must be broken into **small, reversible steps** with clear justification for each.

---

## 📋 PROJECT OVERVIEW

**TOTL Agency** – Talent booking platform connecting models/actors with casting directors and brands.

**Tech Stack:**

- Next.js 15.2.4 + App Router + TypeScript 5
- Supabase (PostgreSQL + Auth + Storage + Real-time)
- TailwindCSS + shadcn/ui components
- Resend API for custom emails

**Database:** PostgreSQL with RLS on all tables  
**Auth:** Supabase Auth with role-based access (talent/client/admin)  
**Key Tables:** `profiles`, `talent_profiles`, `client_profiles`, `gigs`, `applications`, `bookings`, `portfolio_items`

---

## 📋 PROJECT CONTEXT SUMMARY

- **Tech:** Next.js App Router + Supabase + TypeScript + shadcn/ui
- **Database:** PostgreSQL with strict RLS
- **Auth:** Role-based (`talent`, `client`, `admin`) tied to `profiles`
- **Security:** RLS enforced everywhere, no service keys in client
- **Architecture:** Server components for data fetching; presentational components only on the client

---

## ✅ COMPLIANCE CHECKLIST (BEFORE GENERATING CODE)

Before writing or editing any code, confirm:

- [ ] Read `TOTL_PROJECT_CONTEXT_PROMPT.md` for full context
- [ ] `database_schema_audit.md` is treated as a **schema audit** that must match `supabase/migrations/**` + generated `types/database.ts`
- [ ] Used proper TypeScript types (no `any`)
- [ ] Followed RLS-compatible query patterns
- [ ] Kept database logic **out of React components**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Digital-Builders-757) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
