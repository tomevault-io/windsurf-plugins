---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # start Vite dev server (localhost:5173)
npm run build    # tsc type-check + Vite production build → dist/
npm run preview  # serve the dist/ folder locally
```

No test suite is configured. Type correctness is the primary correctness mechanism — run `npx tsc --noEmit` for a standalone type check.

## Architecture

**Steuerhelfer** is a fully client-side React SPA for Austrian tax return preparation (Arbeitnehmerveranlagung). All data lives in `localStorage` under the key `steuerhelfer-at-v1` — there is no backend.

### State & data flow

`TaxContext` ([src/context/TaxContext.tsx](src/context/TaxContext.tsx)) is the single source of truth. It uses `useReducer` with a typed `Action` union and auto-persists to `localStorage` on every state change. The state shape is:

```
State
  selectedYear: number          ← which tax year the user is viewing
  years: TaxYearData[]          ← one entry per year, created on demand
```

Each `TaxYearData` contains: `afaAssets`, `werbungskosten`, and the per-year `otherBroker*` fields. Pages read `selectedYearData` (the pre-selected slice) from context rather than filtering themselves.

`State` also carries a top-level **`portfolioEvents: PortfolioEvent[]`** — a single, cross-year list of every share movement (RSU `VEST`, ESPP `BUY`, `EXERCISE`, `SELL`). This is **not** partitioned by year because the Austrian moving-average cost basis (Gleitender Durchschnittspreis, §27a EStG) is a running figure across the entire holding period: a sale's gain depends on every acquisition that preceded it. Mutated via `ADD_PORTFOLIO_EVENTS` / `DELETE_PORTFOLIO_EVENT` / `DELETE_ALL_PORTFOLIO_EVENTS`.

`LanguageContext` ([src/context/LanguageContext.tsx](src/context/LanguageContext.tsx)) wraps `react-intl`'s `IntlProvider`. All user-facing strings must use `useIntl().formatMessage({ id: '...' })` with keys defined in [src/i18n/de.ts](src/i18n/de.ts) and [src/i18n/en.ts](src/i18n/en.ts). Add new keys to both files.

### Pages (routes under `<Layout>`)

Routes are defined in [src/App.tsx](src/App.tsx); the index route (`/`) renders the FinanzOnline guide — there is no separate dashboard.

| Route | File | Purpose |
|---|---|---|
| `/` (index) | [src/pages/FinanzOnlineGuide.tsx](src/pages/FinanzOnlineGuide.tsx) | Step-by-step FinanzOnline filing guide + computed KZ summary for the selected year |
| `/etrade` | [src/pages/EtradeSection.tsx](src/pages/EtradeSection.tsx) | Portfolio ledger (vests/buys/sells) + moving-average gains; imports eTrade files |
| `/afa` | [src/pages/AfaCalculator.tsx](src/pages/AfaCalculator.tsx) | Depreciation (AfA) asset management |
| `/werbungskosten` | [src/pages/WerbungskostenSection.tsx](src/pages/WerbungskostenSection.tsx) | Employee expense deductions |

### Tax calculations ([src/utils/calculations.ts](src/utils/calculations.ts))

All tax logic lives here. Key functions:

- `calculateAfa(asset, forYear)` — implements §16 Abs. 1 Z 8 lit. b EStG half-year rule: H1 purchase → full annual AfA; H2 purchase → half first year + extra trailing half year. GWG (≤ €1 000) → immediate full deduction. Supports linear and degressive (30%) methods with automatic switch-to-linear.
- `calculateWerbungskosten(data, totalAfaDeductions)` — applies Austrian statutory limits (ergonomic furniture capped at €300/year including prior-year carry-over).
- `calculateCapitalGains(portfolioEvents, year)` — runs the moving-average engine over the **whole** event history, then slices out the realized sells and vests that fall in `year`. Returns the year's gains/losses, the realized-sale entries (`CapitalGainEntry`), and the full engine ledger.
- `calculateTaxSummary(data, portfolioEvents)` — assembles the complete `TaxSummary` with all KZ fields (KZ 158, 169, 277, 717, 720, 721, 722, 724, 994). KZ 994/892 now come from `calculateCapitalGains`, not per-sale cost bases.

`TaxSummary` fields map 1:1 to FinanzOnline Kennzahlen (KZ). When adding a new deduction category, add it to `TaxSummary`, compute it in `calculateTaxSummary`, and surface it in [FinanzOnlineGuide](src/pages/FinanzOnlineGuide.tsx) / [TaxSummaryPrint](src/components/TaxSummaryPrint.tsx).

### Moving-average engine ([src/utils/taxEngine.ts](src/utils/taxEngine.ts))

`runTaxEngine(events)` implements the Austrian moving-average cost basis (Gleitender Durchschnittspreis, §27a EStG) — a faithful TS port of the `tax-etrade` Python `TaxEngine`. Events are sorted by date with acquisitions before sells on the same day (VEST=0, BUY/EXERCISE=1, SELL=2, for sell-to-cover). Acquisitions recompute `avg = (oldTotalCost + newCost) / (oldShares + newShares)`; sells realize `(sellPriceEUR − avg) × shares` and leave the average unchanged; a depot check flags selling more than held. All money is rounded to 4 decimals half-away-from-zero to match the Python `Decimal` behaviour. Verified to produce identical per-year gains/losses to the reference tool.

### eTrade import ([src/utils/parseEtradeFiles.ts](src/utils/parseEtradeFiles.ts))


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Southgarden116/austrian-tax-helper](https://github.com/Southgarden116/austrian-tax-helper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
