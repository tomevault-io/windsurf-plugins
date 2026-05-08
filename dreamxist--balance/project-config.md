---
trigger: always_on
description: App de finanzas personales con balance assertion y reconciliacion. Monorepo TypeScript.
---

# Balance — Personal Finance App

## Overview
App de finanzas personales con balance assertion y reconciliacion. Monorepo TypeScript.

## Architecture
- Monorepo: npm workspaces + turborepo
- packages/core: shared business logic (TypeScript)
- apps/web: Vite + React SPA (TanStack Router/Query, Tailwind v4)
- apps/cli: CLI tool `bal` (commander)
- supabase/: migrations, functions, seed data
- Backend: Supabase (PostgreSQL + RLS + Database Functions)
- No custom backend server

## Key docs
- docs/architecture.md — DB schema, functions, security, folder structure
- docs/workflows.md — User flows, business rules, financial logic
- docs/design.md — UI patterns, wireframes, color system, typography
- data/excel-analysis.md — Original Excel analysis (reference only)

## Commands
```bash
npm install              # install all deps
npm run dev              # start web + supabase local
npm run dev:web          # start web only
npm run dev:cli          # run CLI in dev mode
supabase start           # start local Supabase
supabase db reset        # reset + seed local DB
supabase gen types typescript --local > packages/core/src/types.ts
```

## Code conventions
- TypeScript strict, no `any`
- Named exports only
- Prefer `function` declarations over arrow for top-level
- Business logic lives in packages/core, NEVER in apps/web or apps/cli
- Database logic lives in supabase/migrations as PL/pgSQL functions
- UI components in apps/web/src/components/{feature}/
- All money amounts stored as integers (CLP in pesos, USD in cents)
- Commit messages in English

## Financial model (critical)
The app uses Balance Assertion with Reconciliation:
- Position = sum(on_budget assets) - sum(on_budget liabilities)
- Accumulated = sum(income + expense + refund + adjustment transactions)
- Delta = Position - Accumulated (should be 0)
- Transfers and debt_payments do NOT affect accumulated
- Installment purchases: full amount as expense at purchase time, monthly payments are debt_payments
- See docs/workflows.md for complete flow details

## Database patterns
- ALL business logic in Database Functions (PL/pgSQL), not in application code
- Functions decomposed: _primitives (SQL) -> operations (PL/pgSQL) -> views (read-only)
- RLS on every table, deny by default
- Use `(select auth.uid())` in RLS policies (cached, not per-row)
- Views MUST have `security_invoker = true`
- Transactions are immutable — correct with undo/refund, never update/delete
- Snapshots are immutable
- Soft delete via is_archived, never hard delete on financial tables
- CLI uses user JWT (from `bal login` or API key), NOT service_role
- API keys validated via Edge Function, generate short-lived JWT
- service_role only in Edge Functions and admin scripts

## Testing
- Database functions: pgTAP tests in supabase/tests/
- Core logic: vitest in packages/core/
- Web components: vitest + testing-library in apps/web/
- RLS policies: dedicated pgTAP tests per table

## Don't
- Don't add ORMs (Prisma, Drizzle) — use supabase-js + generated types
- Don't add global state (Redux, Zustand) — TanStack Query handles server state
- Don't put business logic in React components — extract to packages/core
- Don't create API routes — use Supabase RPC directly
- Don't store money as floats — always integers
- Don't delete financial records — archive or create reversals

<!-- GSD:project-start source:PROJECT.md -->
## Project

**Balance**

App de finanzas personales que unifica la gestion financiera de una persona y una segunda entidad opcional (SpA u otra empresa) en una sola plataforma. El patron core es Balance Assertion with Reconciliation: cada peso esta ubicado y explicado, delta = 0.

**Core Value:** Cuadrar el balance — que la posicion financiera real (saldos en cuentas) coincida con el registro de movimientos (transacciones acumuladas). Delta = 0.

### Constraints

- **Stack**: Vite + React (no Next.js) — SPA estatico, deploy en Vercel
- **Backend**: Supabase only — no custom backend server
- **DB Logic**: Toda logica de negocio en Database Functions (PL/pgSQL), no en aplicacion
- **Auth**: JWT via Supabase Auth + API keys via Edge Function — CLI no usa service_role
- **Money**: Integers (CLP en pesos, USD en centavos), nunca floats
- **Immutability**: Transactions y snapshots inmutables — corregir con undo/refund
- **RLS**: Deny by default, `(select auth.uid())` pattern, security_invoker en views
- **Single user v1**: Pero RLS y auth preparados para multi-user futuro
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->
## Technology Stack

## Validation Summary
| Area | Veredicto | Accion |
|------|-----------|--------|
| Vite 6 | **ACTUALIZAR a Vite 8** | Vite 8.0.3 es estable, usa Rolldown (10-30x builds mas rapidos) |
| React 19 | CONFIRMADO | Estable, sin issues |
| Supabase (supabase-js v2) | CONFIRMADO con **FLAG** | Migrar a nuevo modelo de API keys (publishable/secret) durante desarrollo |
| TanStack Router | CONFIRMADO | v1.168+, excelente con Vite y React 19 |
| TanStack Query | CONFIRMADO | Sigue siendo la eleccion correcta |
| Tailwind v4 | CONFIRMADO | CSS-first config funciona bien |
| shadcn/ui | CONFIRMADO con ajustes | Usar `tw-animate-css` en vez de `tailwindcss-animate` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dreamxist/balance](https://github.com/dreamxist/balance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
