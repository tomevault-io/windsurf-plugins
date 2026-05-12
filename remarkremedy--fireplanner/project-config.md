---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Singapore FIRE (Financial Independence, Retire Early) + Property + Investment Retirement Planner. A fully client-side web application for comprehensive retirement planning tailored to Singapore residents.

**Status:** Shipped. Fully client-side app deployed to Cloudflare Pages. Monte Carlo simulation (Web Worker), 12 withdrawal strategies, historical backtesting, sequence risk stress testing, dashboard, property analysis, scenario save/load, Excel/JSON export, and reference guide.

**Remaining gaps:**
- Property hybrid MC overlay (5 discrete weighted property scenarios per MC path)
- Scenario side-by-side comparison view (save/load works, parallel compare panel does not exist)

**Deployment:** Cloudflare Pages via `wrangler pages deploy`. No server. Sentry and PostHog intentionally skipped (privacy-first, no-server-contact promise).

**Source of truth:** When the master plan (`FIRE_PLANNER_MASTER_PLAN_v2.md`) and this file conflict, the master plan wins for calculation logic, formulas, Singapore-specific rules, and domain requirements. This file wins for technology choices, architecture decisions, and implementation patterns.

## Architecture

### Frontend
| Dependency | Version |
|-----------|---------|
| React | 19.x |
| TypeScript | 5.9.x |
| Vite | 7.x |
| React Router | 6.x |
| Zustand | 5.x |
| React Query (TanStack Query) | 5.x |
| Tailwind CSS | 3.4.x |
| shadcn/ui | latest |
| Recharts | 2.x |
| D3.js | 7.x |
| Zod | 3.x |
| TanStack Table | 8.x |

**Routing:** React Router v6 with `createBrowserRouter`. Route components live in `pages/` as plain components (e.g., `InputsPage.tsx`). Do NOT use Next.js file-based routing conventions (`page.tsx`, `layout.tsx` in nested folders).

**State:** 7 Zustand stores (profile, income, allocation, simulation, withdrawal, property, ui). Dashboard metrics are **derived hooks**, not a store — the dashboard owns no state, it computes views from other stores.

### Simulation Engine (Web Worker)

All heavy computation (Monte Carlo, backtest, sequence risk, SWR optimization) runs in a Web Worker (`lib/simulation/simulation.worker.ts`) to avoid blocking the UI. No backend server required.

| Module | Purpose |
|--------|---------|
| `lib/math/linalg.ts` | Cholesky decomposition, covariance matrix, matrix ops |
| `lib/math/random.ts` | SeededRNG (xoshiro128**), Box-Muller gaussians |
| `lib/math/stats.ts` | Percentile, Student-t quantile |
| `lib/simulation/monteCarlo.ts` | 10K MC simulations (parametric/bootstrap/fat-tail) |
| `lib/simulation/backtest.ts` | Bengen-style rolling window historical backtest |
| `lib/simulation/sequenceRisk.ts` | Crisis scenario stress testing |
| `lib/simulation/swrOptimizer.ts` | Binary search for safe withdrawal rate |
| `lib/simulation/simulation.worker.ts` | Web Worker message handler |
| `lib/simulation/workerClient.ts` | Worker client + strategy params flattening |
| `lib/exportExcel.ts` | Client-side Excel export via exceljs |
| `lib/data/historicalReturnsFull.ts` | 98 rows historical returns (1928-2025) |

### Data Persistence (Browser-Only)

All user financial data stays in the browser. No server-side storage of user data.

- **localStorage:** All Zustand store state auto-saved on change via `zustand/middleware` persist. Restored on page load.
- **JSON export/import:** Users can download their full state as a JSON file and restore it on any device/browser. This is the cross-device portability mechanism.
- **URL params:** Life stage, FIRE type, and view state encoded in URL for bookmarking and sharing specific views.
- **No authentication.** No accounts. No server-side anything.

### Key Directories
```
frontend/src/
├── pages/          # Route components: StartPage, InputsPage, ProjectionPage,
│                   #   WithdrawalPage, StressTestPage, DashboardPage,
│                   #   ChecklistPage, ReferencePage
├── components/     # UI by domain: ui/, layout/, profile/, income/, allocation/,
│                   #   simulation/, withdrawal/, backtest/, dashboard/, property/,
│                   #   cpf/, goals/, healthcare/, projection/, sequenceRisk/, shared/
├── stores/         # 7 Zustand stores (profile, income, allocation, simulation,
│                   #   withdrawal, property, ui)
├── hooks/          # ~22 derived hooks (FIRE calcs, projections, dashboard, etc.)
└── lib/
    ├── calculations/  # fire, cpf, tax, income, portfolio, withdrawal, property,
    │                  #   srs, hdb, healthcare, expenses, projection, cashReserve, timeCost
    ├── simulation/    # monteCarlo, backtest, sequenceRisk, swrOptimizer, worker
    ├── math/          # linalg, random, stats
    ├── validation/    # schemas, rules
    └── data/          # historicalReturns, cpfRates, taxBrackets, momSalary,
                       #   stampDutyRates, healthcarePremiums, hdbRates, balaTable,
                       #   crisisScenarios, sources, changelog, goalTemplates, etc.
```

Use `Glob` or `ls` to discover specific files. The router (`router.tsx`) defines all routes; legacy paths like `/profile`, `/income`, `/monte-carlo` redirect to consolidated pages.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RemarkRemedy/fireplanner](https://github.com/RemarkRemedy/fireplanner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
