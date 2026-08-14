---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

LMU Analyzer — a client-side web app for analyzing Le Mans Ultimate (sim racing) XML race data. Users select a local folder or upload XML files; all parsing and visualization happens in the browser with zero server communication.

Built by Armin Reiter (axrider) at a31 Labs. Deployed to Cloudflare Pages.

## Commands

```bash
pnpm run dev          # Start dev server (http://localhost:5173)
pnpm run build        # TypeScript check + Vite production build
pnpm run lint         # ESLint
pnpm run preview      # Preview production build locally
```

Build outputs to `dist/`. Deployed via Cloudflare Pages (`wrangler.jsonc`).

## Architecture

**Pure client-side app** — no backend, no database, no API calls. Data stays in the browser.

### Data Flow

1. User selects folder (File System Access API) or uploads files (fallback for Brave/Firefox)
2. `src/lib/parser.ts` — Parses rFactor XML format into typed `RaceFile[]` structures
3. `src/lib/analytics.ts` — Computes stats, personal bests, race results from parsed data
4. `src/lib/storage.ts` — Persists parsed data to IndexedDB, filters/preferences to localStorage
5. `src/App.tsx` — Holds state: `files`, `selectedDrivers`, `selectedClasses`; applies class filter via `filterFilesByClasses()` before passing `filteredFiles` to views

### Global Filters (in Header)

- **Driver multi-select** — filters by driver name across all views (shows session/lap count badges per driver)
- **Car class multi-select** — filters entire dataset by class (Hyper/GT3/GTE/LMP3); applied at App level before data reaches views
- **Benchmark toggle** — enables/disables community pace comparison (Race Pace view)

### Key Files

- `src/lib/types.ts` — All TypeScript types (`RaceFile`, `SessionData`, `DriverResult`, `LapData`, `PersonalBest`, `CarClass`, etc.)
- `src/lib/parser.ts` — XML→TypeScript parser (`loadFolder()`, `parseUploadedFiles()`, both return `{ files, failedFiles }`). Handles session types (Practice/Qualifying/Race/Warmup), lap data with sectors/tire wear/fuel, incidents, penalties, track limits
- `src/lib/analytics.ts` — All data computations. Functions accept `driverNames: string | string[]` to support multi-driver selection. Key functions: `getOverviewStats()`, `getPersonalBests()`, `getAllSessionBests()`, `getTheoreticalBest()`, `getTrackStats()`, `getCarStats()`, `detectPlayerDrivers()`, plus shared predicates `isDnf()`, `isValidLap()`, `isOnline()`, `isRatedRace()`. Includes `deduplicateSessions()` to merge duplicate Race sessions (matching timestamp, track, and source tag)
- `src/lib/storage.ts` — Persistence layer: IndexedDB for parsed files (versioned cache) + FileSystemDirectoryHandle (refresh from folder), localStorage for filter preferences and profile via exported `KEYS` registry (all localStorage keys must be registered there so `clearAll()` sees them). Graceful fallback if IndexedDB unavailable
- `src/lib/racepace.ts` — Community benchmark integration. Fetches pace tiers from ohne_speed's Google Sheet CSV. Rates laps as Alien/Competitive/Good/Midpack/Tail-ender/Offline. Single source of truth for tier order (`RATING_ORDER`), tier colors, and benchmark lookup (`buildBenchmarkMap()`)
- `src/lib/DataIndexContext.tsx` + `dataIndexStore.ts` + `useDataIndex.ts` — Context providing precomputed indices over the loaded files (per-driver sessions, all laps, sector minimums for theoretical bests)
- `src/lib/useBenchmarks.ts` — Hook wrapping the benchmark fetch (returns `benchmarks`, `benchmarkMap`, `loading`, `error`)
- `src/lib/sessionContext.ts` — `buildSessionContext()`/`parseSessionContext()` for the session-navigation string
- `src/lib/formatting.ts` — Time/number formatters, `errorMessage()`, chart theme constants (`CHART_AXIS_TICK`, `CHART_GRID_STROKE`)
- `src/lib/useInstallPrompt.ts` — PWA install prompt hook; `useTheme.ts` — light/dark theme; `useClickOutside.ts` — shared outside-click hook (use this, don't hand-roll listeners)

### Components

- `src/components/Header.tsx` — Sticky header: logo, driver/class filters, benchmark toggle, refresh/install/reload buttons, tab navigation
- `src/components/FolderPicker.tsx` — Initial file loading UI: folder selection (FSA), file upload fallback, resume cached data
- `src/components/SortableTable.tsx` — Reusable sortable table with fixed column widths via `<colgroup>`. All data tables use this component
- `src/components/SearchableMultiSelect.tsx` — Multi-select dropdown with search
- `src/components/SearchableSelect.tsx` — Single-select dropdown with search
- `src/components/ClassBadge.tsx` — Color-coded car class badge
- `src/components/StatCard.tsx` — Stat widget (label, value, icon, subtext) with variants `default`/`tile`/`center`; use these instead of hand-built stat divs
- `src/components/ExportButton.tsx` — Export table data as CSV/XLSX (slugifies the `filename` prop itself — pass raw names)
- `src/components/Footer.tsx` — Build time, links, copyright
- `src/components/DataCardHeader.tsx` — Standard card header (title, actions slot) used by all data cards
- `src/components/RatingBadge.tsx` — Pace-tier badge (sizes `sm`/`md`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arminreiter/lmu-analyzer](https://github.com/arminreiter/lmu-analyzer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
