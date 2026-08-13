---
trigger: always_on
description: Architecture and conventions reference for AI-assisted development.
---

# CLAUDE.md — LittleFlower / Florecita

Architecture and conventions reference for AI-assisted development.

---

## Project overview

Florecita is an offline-first household budget tracker for couples. Frontend-only deployment (Vercel), with PowerSync for local SQLite + sync and Supabase as the Postgres backend and auth provider.

**Live domain:** `https://florecita.club`
**Total infra cost:** $0/month (all free tiers)

---

## Stack

| Concern | Technology |
|---|---|
| UI | React 19 + TypeScript + Vite |
| Routing | TanStack Router (file-based, `src/routeTree.gen.ts` auto-generated) |
| Tables | TanStack Table |
| Local storage | PowerSync — SQLite WASM via `@journeyapps/wa-sqlite` |
| Sync + auth backend | Supabase (Postgres + `@supabase/supabase-js`) |
| UI components | shadcn/ui + Tailwind CSS (pastel pink/violet theme) |
| Error tracking | Sentry |
| PWA | vite-plugin-pwa + Workbox |

---

## Architecture

Strict DDD layering — inner layers have zero dependencies on outer layers.

```
domain/          ← pure types + business logic, no imports from other layers
application/     ← composable hooks: query hooks + domain calculations
infrastructure/  ← PowerSync repos, raw SQL query hooks, Supabase connector
features/        ← UI (pages + components). Imports from application/ only.
shared/          ← design system, utilities, i18n context
```

### Hard rules

- **No `useQuery` in `features/`** — all reactive SQL reads go through `application/` hooks.
- **No business calculations in `features/`** — income conversion, balance calc, etc. live in `application/`.
- **All mutations go through `mutationWrappers.ts`** — never call `db.execute()` directly from features. Wrappers auto-populate `last_edited_by` + `last_edited_at` and return the new record ID.

---

## PowerSync integration

### Key files

| File | Purpose |
|---|---|
| `src/infrastructure/powersync/schema.ts` | PowerSync SQLite schema (source of truth for local tables) |
| `src/infrastructure/powersync/client.ts` | Exports `db` singleton |
| `src/infrastructure/powersync/SupabaseConnector.ts` | Connector + exports `supabase` client |
| `src/infrastructure/powersync/mutationWrappers.ts` | All write operations |
| `src/infrastructure/powersync/repositories/` | Domain repositories |
| `src/infrastructure/powersync/queries/` | All raw SQL strings (PowerSync `useQuery` hooks) |

### Reactive query layer

All raw SQL strings live in `src/infrastructure/powersync/queries/` — never inline SQL in features or application hooks.

```
queries/
  useBudgetQueries.ts
  useExpenseQueries.ts
  useIncomeQueries.ts
  useExchangeRateQueries.ts
  useDebtQueries.ts
  useHouseholdQueries.ts
  types.ts      ← canonical SQLite row types (snake_case)
  index.ts      ← barrel export
```

Application hooks in `src/application/` compose these query hooks with domain calculations:

```
useAppShellData.ts      ← budget table + income metrics for AppShell
useDashboardData.ts     ← 6-month aggregate data for all dashboard charts
useIncomeSummary.ts     ← income total with TRM conversion
useDebtSummary.ts       ← debt balances (management + dashboard variants)
useMonthViewData.ts     ← all data for MonthViewPage
useYearViewData.ts      ← all data for YearView (spreadsheet)
```

### Lifecycle

- `db.connect(new SupabaseConnector())` — called after Supabase login
- `db.disconnect()` — called on logout
- WASM requires COOP/COEP headers (`vite.config.ts` + `vercel.json`)
- `optimizeDeps.exclude: ['@powersync/web']` in `vite.config.ts` — prevents Vite pre-bundling WASM

### Checklist for adding a new table

1. Migration SQL in Supabase (table + RLS policy with `household_id`)
2. Table entry in `schema.ts`
3. **Sync Rules in PowerSync Cloud dashboard** → `SELECT * FROM <table> WHERE household_id = bucket.household_id` — without this, data never reaches the client
4. Repository in `src/infrastructure/powersync/repositories/`
5. Mutations in `mutationWrappers.ts`

### Gotcha: `rowsAffected` unreliable on PowerSync views

PowerSync exposes local tables as **views** (underlying storage is `ps_data__*` tables). `result.rowsAffected` from `UPDATE` / `DELETE` on a view is **NOT reliable** — it can return 0 even when the statement succeeded.

**Do NOT use this pattern for upserts:**
```typescript
// BROKEN: rowsAffected is unreliable on views
await db.execute('UPDATE ... WHERE id = ?', [id]);
if ((result.rowsAffected ?? 0) > 0) return id;
// falls through to INSERT with same id → UNIQUE constraint violation!
```

**Use explicit `SELECT EXISTS` instead:**
```typescript
// CORRECT: verify existence explicitly
const exists = !!(await db.get('SELECT id FROM table WHERE id = ?', [id]));
if (exists) {
  await db.execute('UPDATE ... WHERE id = ?', [id]);
  return id;
}
// safe INSERT here
```

Affected functions: `upsertBudgetMonth`, `upsertIncomeMonth`, `upsertExchangeRate`.

---

## Domain model

```
Budget (1) ──── (N) BudgetMonth
Income (1) ──── (N) IncomeMonth
BudgetMonth (1) ── (N) Expense
ExchangeRate   ← per month, USD→COP TRM
Household      ← 6-digit join code for onboarding
```

### Expense rules

Expenses are an **append-only ledger** — no edits, no deletes in the traditional sense:

- **Deletion** = new negative record (`amount < 0`, `linked_expense_id` = original)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ElArabeLoco/LittleFlower](https://github.com/ElArabeLoco/LittleFlower) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
