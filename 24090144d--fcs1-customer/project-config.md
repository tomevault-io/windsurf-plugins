---
trigger: always_on
description: Shared instruction set for all AI coding agents (Claude Code, Codex, Gemini CLI, etc.).
---

# FCS1 Customer — Agent Guide

Shared instruction set for all AI coding agents (Claude Code, Codex, Gemini CLI, etc.).
Read this before writing any code. See `CLAUDE.md` for deep technical patterns (Highcharts/i18n/config-panel details) — still agent-agnostic despite the filename. This project's primary day-to-day development has moved to Codex; this file (`AGENTS.md`) is the authoritative shared starting point for any agent.

---

## Current State

| Key | Value |
|---|---|
| Version | **v1.1.31** (released 2026-07-24) |
| Branch | `main` |
| Local dev | `npm run dev` → `http://localhost:3010` |
| Previous version | v1.1.30 |

**Local-only testing rule:** only test against localhost (`npm run dev`, port 3010). Never push, deploy, or commit unless the user explicitly asks in that turn — a past approval is not standing permission.

---

## Start Here

- Read this file (`AGENTS.md`) first, regardless of agent.
- Read `CLAUDE.md` for deep technical patterns (Highcharts drilldown gotchas, i18n conventions, config-panel wiring).
- Read `docs/co-dev-handoff.md` when resuming CO-specific work.
- Use `main` as the only release branch unless the user explicitly requests a temporary branch or worktree.
- No Claude-specific browser-preview tooling exists outside Claude Code. To verify UI changes in another agent: start the dev server (`npm run dev`), open `http://localhost:3010` in a regular browser, and manually navigate the sidebar (chain → module → hotel/corp) to the affected view. `npx tsc --noEmit` remains the authoritative compile gate on Windows regardless of agent (`next build` fails locally on Windows due to an unrelated symlink issue — see CLAUDE.md).

---

## Section Structure: KPI / Simple Charts / Long Charts (+ Scope-Specific Tables)

All four modules (CO first, then JO/MO/IM) share these three base sections, in this order:

1. **KPI** — `kpi-grid mt-0 grid grid-cols-2 sm:grid-cols-3 md:grid-cols-5 gap-3`
2. **Simple Charts** — `chart-grid mt-5 grid grid-cols-1 md:grid-cols-2 gap-4` (2 per row) — everything lives here by default
3. **Long Charts** — `chart-grid-long mt-5 grid grid-cols-1 gap-4` (1 per row) — reserved for deep multi-level drilldowns; membership is opt-in per chart id, moved in only on explicit request

Explicit scope-specific analytical sections may follow Long Charts. Corp IM has `Hotel → Department → Category → Incident → Detail`; Corp JO has `Hotel → Department → Category → Service Item → Detail`; Corp MO has `Hotel → Department → Category → Defect → Detail`; and Corp CO has `Hotel → Cleaning Type → Stay Status → Attendant → Detail`. Hotel variants omit the Hotel level. The corresponding live routes are `/api/dashboard/im-table`, `/api/dashboard/jo-table`, `/api/dashboard/mo-table`, and `/api/dashboard/co-table`. The first level renders inline, deeper levels use a document-level modal, and every level has an icon-only CSV export. Keep each table date/hotel-filter aware and module-independent.

Each section is headed by a shared `SectionHead` component (label + horizontal rule). **`SectionHead` must include `mb-3`** on its wrapping div — this is the only thing separating the label from the cards below (grids use `mt-0`/`mt-5`, not a top margin). `CoDashboardView.tsx` and `DashboardClient.tsx` each keep their **own copy** of `SectionHead` — if you touch one, mirror the change in the other.

Long-Charts membership lives in per-module `Set<string>` constants (`MO_LONG_CHART_IDS`, `JO_LONG_CHART_IDS`, `IM_LONG_CHART_IDS` in `DashboardClient.tsx`; `LONG_CHART_IDS` in `CoDashboardView.tsx`), all currently empty. Move a chart id into the set + bump its list cap to `N = 50` only when the user names that specific chart — never batch-move charts speculatively.

---

## Module Scope

| Code | Full Name |
|---|---|
| `IM` | Incident Management |
| `JO` | Job Order |
| `MO` | Maintenance Order |
| `CO` | Cleaning Order / ACSR |

---

## Recent Version History

| Version | Date | Summary |
|---|---|---|
| **v1.1.31** | 2026-07-24 | Extended Hotel daily drilldown tables with a business grouping before each rank distribution: JO starts at Department, MO at Department, IM at Department, CO-ACSR at Cleaning Type, and CO-IR at Inspection Status. Group filters now persist through item, daily, detail, breadcrumb, and CSV export levels while Corp hierarchies remain hotel-first. |
| **v1.1.30** | 2026-07-24 | Removed the CO-IR `Duration Source` column from Corp and Hotel daily-inspector detail tables and CSV exports. Hardened the shared terminal detail renderer against PostgreSQL numeric values returned as strings, preventing client-side exceptions when formatting inspection score, credit, duration, standard, and variance. |
| **v1.1.29** | 2026-07-23 | Added live database-backed daily drilldown tables across Corp and Hotel JO, MO, IM, CO-ACSR, and CO-IR. New table routes provide rank distributions, named item/attendant/inspector drilldowns, ascending daily summaries, compact source-record details, CSV export, active dashboard filters, and literal source timestamp preservation without requiring CSV re-upload. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [24090144d/fcs1-customer](https://github.com/24090144d/fcs1-customer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
