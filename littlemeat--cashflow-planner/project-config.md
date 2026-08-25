---
trigger: always_on
description: Personal long-horizon cashflow planner for a Czech household planning maternity leave and a mortgage. 30-year stress-test of finances with stackable life events, salary changes, inflation, mortgage rate changes.
---

# Cashflow Planner — Agent Onboarding

Personal long-horizon cashflow planner for a Czech household planning maternity leave and a mortgage. 30-year stress-test of finances with stackable life events, salary changes, inflation, mortgage rate changes.

## Stack

Vite + React 19 + TypeScript + Tailwind CSS v4 + Zustand + Recharts + LocalStorage. No backend, no auth.

## Project structure

```
src/
  types/index.ts          — all TypeScript interfaces (Plan, Baseline, CashflowEvent, Mortgage, Asset, MonthlySnapshot, Period)
  store/usePlanStore.ts   — Zustand store + localStorage persistence + migratePlan()
  lib/
    simulate/
      index.ts            — simulate(plan): MonthlySnapshot[], getMonthDetail(plan, month): MonthDetail,
                            stepMonth() (shared core step), applyEvents(), computeAssetsValue()
      mortgage.ts         — MortgageState, initMortgageState(), stepMortgage()
      __tests__/          — 50 tests across 4 files (simulate, presets, phase4, hidden)
    formatters.ts         — formatCZK, formatRunway, formatPercent, dateToMonthOffset, monthOffsetToDate,
                            addMonths, formatYearMonth, formatFrequency, formatCategory, computeMonthlyPayment
    formatters.test.ts    — 8 tests for formatting utilities (lives at src/lib/, not in __tests__/)
    constants.ts          — Czech 2026 social/tax constants (PPM cap, rodičovský příspěvek, sleva na dítě)
    seedData.ts           — default plan seeded on first run (uses growthSchedule, schemaVersion: 1)
  components/
    AmountInput.tsx       — controlled number input with space thousand separators (cs-CZ)
    CollapsiblePanel.tsx  — shared collapsible card wrapper (title, defaultOpen?, headerRight?, children)
    Modal.tsx             — shared backdrop modal with safe mouseDownTarget close pattern
    InfoTooltip.tsx       — shared SVG info icon + tooltip (text, width?, position?)
    EyeIcon.tsx           — shared eye / eye-slash SVG icon for visibility toggles
    KpiCard.tsx           — single KPI tile used in ResultsSummary
    BaselinePanel.tsx     — start date, cash, investments, yield, safety buffer, horizon
    EventsPanel.tsx       — CRUD + drag-and-drop sort for income/expense events; eye toggle for hidden
    EventForm.tsx         — add/edit event modal (growthSchedule editor, end date)
    MortgagePanel.tsx     — CRUD for mortgages; eye toggle for hidden
    MortgageForm.tsx      — add/edit mortgage modal with live amortization summary
    AssetsPanel.tsx       — CRUD for assets (property, investments); eye toggle for hidden
    AssetForm.tsx         — add/edit asset modal with linked expense/mortgage pickers
    MonthDetailModal.tsx  — per-event income/expense breakdown for a clicked month
    ResultsPanel.tsx      — thin wrapper: <ResultsSummary /> + <ResultsChart /> (kept for backward compat)
    ResultsSummary.tsx    — KPI bar, warning banner, date slider, Runway card
    ResultsChart.tsx      — line chart + yearly/monthly table toggle + CSV export button
    presets/
      PresetPicker.tsx        — dropdown triggering preset wizards
      MaterskaPreset.tsx      — PPM maternity benefit (calculate from income or manual for OSVČ)
      RodicovskaPreset.tsx    — parental allowance (rodičovský příspěvek)
      SlevaNaDitePreset.tsx   — child tax credit with optional scheduled increases
      ZvyseniPlatuPreset.tsx  — salary raise (% change on existing income event)
      InflacniSkok.tsx        — adds a new growthSchedule entry on selected expense events from a chosen date
      NavratDoPrice.tsx       — return-to-work income event
      PrevzetiHypoteky.tsx   — mortgage takeover wizard (one-time cash payment + mortgage, atomic batchUpdate)
  App.tsx — 3-row layout: KPI bar → (baseline | events+mortgages+assets) → chart
```

## Core architecture

### Month offsets
All event/mortgage/asset timing is stored as integer month offsets from `baseline.startDate` ("YYYY-MM"). Never store dates directly. Convert with `dateToMonthOffset(date, startDate)` and `monthOffsetToDate(offset, startDate)`. Use `<input type="month">` in forms and convert on change.

### simulate(plan): MonthlySnapshot[]
Pure function — no side effects, no React dependencies. Called on every plan change. Produces one snapshot per month for `horizonYears * 12 + 1` months.

Core loop: `stepMonth(plan, state, m): StepResult` — extracts one month's computation into a shared function used by both `simulate()` and `getMonthDetail()`. Do not duplicate this logic.

`applyEvents(plan, m)` uses `isActiveAt(period, m)` from `Period` type for half-open interval checks. Items with `hidden: true` are skipped entirely (no income/expense/mortgage/asset contribution).

### growthSchedule on events
Each `CashflowEvent` has `growthSchedule: Array<{ id, fromMonth, rateAnnual }>` — an ordered list of rate periods relative to the event's `startMonth`. Growth compounds across periods: `growthFactor *= (1 + rateAnnual/12)^monthsInPeriod`. This replaces the old single `annualGrowthPct` field. `migratePlan()` handles the conversion from old plans.

### Safety buffer sweep

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [littlemeat/cashflow-planner](https://github.com/littlemeat/cashflow-planner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
