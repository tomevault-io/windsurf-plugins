---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Identity

**Stack:** React 18 + TypeScript + Vite 5 — this is a **Single Page Application (SPA)**.
**Routing:** React Router v6 (all routes in `src/App.tsx`)
**Styling:** Tailwind CSS 3 + shadcn/ui (Radix UI primitives in `src/components/ui/`)
**Server-state:** TanStack React Query 5
**Global state:** Context API only — `AuthContext` (auth + MFA + rate limiting + subscription) and `ModeContext` (user/business toggle)
**Forms:** React Hook Form + Zod — no exceptions
**Backend:** Supabase (PostgreSQL + RLS + Auth + 39 Edge Functions in Deno TypeScript)
**Payments:** Coupon-based access system for premium tiers. No active payment provider — Grow/HYP have been removed.
**Hosting:** Lovable/Netlify CDN
**UI language:** Hebrew-first, full RTL (`lang="he"`, `dir="rtl"`)
**Supabase project:** `pujsopidbejeuqteormi` (region: ap-southeast-1)

**This is NOT a Next.js project.** There is no App Router, no Server Components, no SSR, no `getServerSideProps`, no `getStaticProps`, no file-based routing. Do not apply Next.js patterns.

## Build & Development Commands

```bash
npm run dev              # Start Vite dev server (http://localhost:8080)
npm run build            # Production build
npx tsc --noEmit         # Type check without emitting
npm run lint             # ESLint
npm run test             # Run all unit tests (Vitest)
npm run test:watch       # Watch mode
npm run test:coverage    # Unit tests with V8 coverage report
npm run test:all         # Run unit tests + E2E tests sequentially
npm run test:e2e         # Playwright E2E tests
npm run test:e2e:headed  # E2E with browser visible
npm run test:e2e:debug   # E2E with Playwright inspector
```

**Running a single test:**
```bash
npx vitest run src/lib/sanitize     # Unit: run tests matching a path/name
npx playwright test auth.spec.ts    # E2E: run a specific spec file
```

**Test file locations:**
- Unit tests: colocated in `src/` as `*.test.ts` / `*.test.tsx` (Vitest + jsdom + Testing Library)
- E2E tests: `tests/e2e/*.spec.ts` (Playwright, Chromium + Mobile Safari)
- Test setup (global mocks for jsdom, framer-motion, sonner, router): `src/test/setup.ts`

**Path alias:** `@/` resolves to `src/` (configured in tsconfig + vite + vitest).

**Supabase CLI:**
```bash
npx supabase gen types typescript --linked > src/integrations/supabase/types.ts  # Regenerate DB types
npx supabase db push                                                              # Push migrations
npx supabase functions deploy <name>                                              # Deploy Edge Function
```

---

## Absolute Rules

### Diagnosis Before Action

- Never claim a fix without tracing the root cause to a specific function, line, data state, or policy. "It should work now" is not a valid conclusion.
- Never describe a change as complete unless you have traced the full behavior path — from user action through component → hook → Supabase call → DB → response → UI update.
- Never silently follow documentation when the code disagrees. `SYSTEM_COHERENCE.md` and `README.md` are reference material, not authority. The implemented code is the source of truth. If there is a conflict, note it explicitly and follow the code.
- Never mark an issue resolved based on "the code looks correct." Verify by tracing a real scenario.

### Scope Discipline

- Never rewrite files unrelated to the current task. If you notice a problem in a different file, note it — do not silently fix it.
- Never rename symbols, reorganize imports, or reformat code unless explicitly asked.
- Never add comments, docstrings, or type annotations to code you did not change.
- Never bundle unrelated cleanup into a task branch.

### Security Rules

- Never weaken any security control without a documented replacement in the same change:
  - Do not remove or bypass rate limiting (client `ClientRateLimiter` + server `check_login_rate_limit` RPC)
  - Do not relax RLS policies without verifying all affected access paths
  - Do not disable or skip Cloudflare Turnstile CAPTCHA
  - Do not loosen CSP headers in `vite.config.ts` or `public/_headers`
  - Do not change token storage from `sessionStorage` to `localStorage`
  - Do not remove session timeout logic (30-minute idle, 25-minute warning)
- Never hardcode secrets, API keys, or credentials anywhere. Frontend uses `import.meta.env.VITE_*`. Edge Functions use `Deno.env.get()`. Server-side secrets (`RESEND_API_KEY`, `TURNSTILE_SECRET_KEY`, etc.) must never appear in the frontend bundle.
- Never use the Supabase service role key in frontend code. It belongs only in Edge Functions for privileged operations.
- Never bypass RLS by switching to the service role client from frontend components.
- When integrating a future payment provider, always verify webhook HMAC signatures before processing. No payment provider is currently active.

### Trust Score Isolation — Critical Architectural Rule

The trust system has three strictly isolated layers. Never mix them.

| Layer | Table | Type | Redeemable | Affects Trust Score |
|---|---|---|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elior1324/reviewwise-hub](https://github.com/elior1324/reviewwise-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
