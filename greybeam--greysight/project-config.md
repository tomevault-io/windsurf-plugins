---
trigger: always_on
description: Greysight is an open source Snowflake cost observability tool. Keep changes
---

# Greysight Agent Guidelines

Greysight is an open source Snowflake cost observability tool. Keep changes
small, tested, and tied to requested behavior.

## Quick Reference

Run from the repository root unless noted. All test suites are hermetic —
no Supabase or Snowflake credentials required.

```bash
npm run test                 # web (Vitest) + api (pytest)
npm run test:web             # apps/web only
npm run test:api             # apps/api only
npm run lint                 # eslint (web) + ruff check/format (api)
npm run typecheck            # tsc --noEmit (web only)
npm run dev                  # web :3000 + api :8000, demo mode by default
npx vitest run <file>        # single web test (run from apps/web/)
uv run pytest tests/<file>   # single api test (run from apps/api/)
uv run --directory apps/auto-savings pytest   # automated-savings worker suite
uv run --directory shared/connect pytest      # shared greysight-connect package
```

`npm run test` does not cover the `apps/auto-savings` worker or the
`shared/connect` package — run their `uv` suites separately.

Copy `.env.example` to `.env` for local demo mode — no external services
needed. Run shell commands through `rtk` when it is available.

## Core Concepts

- **Dataset pipeline.** Every dashboard metric flows one path: approved
  read-only SQL in `sql/snowflake/` → registered as a source (or composed
  into a derived dataset) in `sql/dashboard_sources.yml` → computed in
  `apps/api/app/services/cost_metrics.py` → prepared into a dashboard view by
  `apps/api/app/services/dashboard_view_builder.py` → fetched, validated, and
  rendered by `apps/web` dashboard components. Demo mode serves the same
  dataset keys from `apps/api/app/services/demo_data.py`.
- **Prepared dashboard views.** Backend code owns analytics, currency/storage
  pricing, date-window semantics, rankings, projections, and unsupported-state
  decisions. Frontend code owns API fetching, cache/prefetch behavior, view
  contracts, and presentation. Do not add frontend-only analytics transforms;
  if a chart needs new derived numbers, add them to the backend
  `DashboardView` model/builder and mirror the contract in
  `apps/web/src/lib/dashboard-contracts.ts`.
- **Dataset key alignment.** Demo data, Snowflake metrics, and frontend
  dataset keys must stay in sync. A new dataset lands as one change touching
  SQL asset + registry entry + metrics + demo data + frontend + tests.
- **Two modes.** Demo mode (`DATA_SOURCE=demo`, `AUTH_REQUIRED=false`) runs
  with no credentials. Authenticated mode uses Supabase auth with
  organization membership and RLS, and executes registry SQL against
  Snowflake. The demo bypass must never leak into authenticated code paths.
- **Automated Savings control loop (separate from the dashboard pipeline).**
  A standalone opt-in worker (`apps/auto-savings/`, its own Railway service)
  polls `SHOW WAREHOUSES` per tenant and directly suspends an enrolled
  `STANDARD` warehouse on the first snapshot that proves it idle, with valid
  zero running and queued statement counts, zero quiescing percentage, and at
  least 62 seconds of uptime.
  Invariants: the worker is the only Snowflake writer and only issues
  `ALTER WAREHOUSE … SUSPEND`; it never changes `AUTO_SUSPEND` or cluster
  settings. Enrollment captures mandatory timezone-aware `created_on`
  identity, and the worker rechecks the versioned enrollment plus global and
  warehouse switches immediately before each command. See
  `docs/automated-savings.md` for the operational runbook and env knobs.

## Dashboard Design System

- Use Tremor chart assets and the shared dashboard primitives before adding
  custom chart or card styling.
- Extend components in `apps/web/src/components/dashboard/` when a reusable
  pattern is missing; avoid one-off dashboard UI.
- Keep spacing on the 4/8/16/24 scale for dashboard layout and controls.
- Structure dashboard pages as header -> controls -> KPI row -> chart/card
  sections.
- Keep dashboard content in a centered container around `1200px` max width.
- Do not add frontend-only analytics transforms; backend prepared views own
  derived dashboard numbers.
- Avoid custom one-off SVGs and chart styling unless a shared asset cannot
  cover the required behavior.

## Project Structure

- `apps/web/`: Next.js app, dashboard UI, auth/org shell, browser API clients, and Vitest tests.
- `apps/api/`: FastAPI backend, trusted auth/org guards, Snowflake access, metric calculation, route tests, and `uv` config.
- `apps/auto-savings/`: standalone Automated Savings worker (own Railway service + Dockerfile) — per-tenant async poll/authorize/suspend loop; hermetic pytest suite mocking the Snowflake session and store.
- `shared/connect/`: installable `greysight-connect` package (Snowflake/Supabase connection code) consumed by `apps/api` and `apps/auto-savings` via a uv path dependency; thin re-export shims remain at `apps/api/app/services/`.
- `sql/snowflake/`: approved read-only Snowflake Account Usage source queries.
- `sql/dashboard_sources.yml`: registry that maps dashboard dataset keys to approved SQL assets and derived datasets.
- `supabase/migrations/`: Supabase schema, RLS policies, organization membership model, and aggregate dataset tables.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [greybeam/greysight](https://github.com/greybeam/greysight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
