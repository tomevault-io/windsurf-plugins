---
trigger: always_on
description: AI urban intelligence platform - natural language queries over global geospatial data (weather, terrain, buildings, population, places, transportation, land use, addresses). Built on Vite + React Router + Tambo AI + DuckDB-WASM + deck.gl.
---

# Walkthru Earth AI

AI urban intelligence platform - natural language queries over global geospatial data (weather, terrain, buildings, population, places, transportation, land use, addresses). Built on Vite + React Router + Tambo AI + DuckDB-WASM + deck.gl.

## Commands

```bash
pnpm dev          # localhost:5173/ai
pnpm build        # production (base: /ai, output: out/)
pnpm preview      # preview production build
pnpm lint         # biome check
pnpm lint:fix     # biome auto-fix
```

- **Vite** (not Next.js) - `vite.config.ts`: React plugin, `@tailwindcss/vite`, `base: "/ai"`, output to `out/`
- **React Router** - `src/App.tsx` defines routes: `/`, `/chat`, `/explore`, `/interactables`, `/style-editor` (lazy-loaded)
- **Entry point**: `index.html` → `src/main.tsx` → `<BrowserRouter basename="/ai">`
- **Biome** (not ESLint) - `biome.json`: 2-space indent, double quotes, semicolons, 120 chars
- **Pre-commit**: lefthook runs `pnpx @biomejs/biome check --write` on staged files
- **Env vars**: Use `import.meta.env.VITE_*` (not `process.env.NEXT_PUBLIC_*`). Defined in `.env.local`, typed in `vite-env.d.ts`
- **No SSR**: Pure SPA, all pages are client-rendered. No `"use server"`, no API routes
- **Fonts**: Quicksand (local woff2 via `@font-face` in `globals.css`) + DM Mono (`@fontsource/dm-mono` in `main.tsx`), fully offline, no CDN
- **Lazy loading**: Use `React.lazy()` + `<Suspense>` instead of `next/dynamic`
- **Static assets**: `basePath` from `import.meta.env.BASE_URL` (set by Vite `base` config)

## Architecture

**queryId pattern** (zero-token data bridge): AI calls `runSQL` → DuckDB executes → full result stored in `query-store.ts` → only `queryId` returned to LLM (~10 tokens). Components read data from store via `useQueryResult(queryId)`.

**Geometry auto-detection**: `runQuery()` auto-detects geometry columns via `DESCRIBE` (fast, metadata-only). Two paths: (1) native GEOMETRY type → `ST_AsWKB` + `ST_Centroid`, (2) WKB BLOB with well-known column name (geom, geometry, shape, etc.) → `ST_GeomFromWKB` + direct WKB passthrough. `enable_geoparquet_conversion = false` in init prevents WASM `stoi` crash on some GeoParquet files, our wrapping handles geometry instead. WKB arrays stored in query-store → GeoArrow zero-copy rendering. AI just writes `SELECT * FROM parquet_file`. **Synthetic lat/lng**: When geometry is auto-detected, `lat`/`lng` columns are injected into results, they do NOT exist in the raw Parquet file. `runQuery` returns a `geometryNote` field explaining which column holds the actual geometry. AI must use `SELECT *` (auto-wrapping re-generates lat/lng). Never reference `lat`/`lng` directly on the raw file.

**Cross-filter bus**: Lightweight pub/sub in `query-store.ts`. Components emit/consume `bbox` (map viewport) and `value` (click) filters. Requires shared `queryId` + `hex`/`pentagon` column. **Time filter bus**: `setTimeFilter()` / `useTimeFilter()` / `applyTimeFilter()` for timestamp-based cross-filtering (TimeSlider → GeoMap snapshot + Graph reference line). **Panel restore bus**: `requestRestorePanel()` / `consumeRestoreRequest()` / `useRestoreVersion()` for restoring dismissed panels. **Emit guards**: `setTimeFilter` and `setCrossFilter` value-check against the current state and skip emit when identical, preventing redundant subscriber re-renders that cascade into React error #185 (infinite update loop) when many panels read the same bus.

**Dashboard canvas**: Desktop = `react-grid-layout`, Touch = `@dnd-kit/sortable` (1.2s hold, grip-only drag). Panel IDs are deduplicated via `Set`. State persisted to localStorage per thread: panel order (`panel-order-${threadId}`), panel layouts/sizes (`panel-layouts-${threadId}`, debounced 500ms), dismissed panels (`panel-dismissed-${threadId}`). First panel and maps always full-width. Non-first/non-map panels pair in 2-column layout. `nonFullCount` counter tracks column position (resets after full-width panels). **Panel sizing**: `panelHeight()` returns grid rows per type: maps=10 (2× other panels), graphs=5, tables=5, QueryDisplay/InsightCard/DatasetCard=3, StatsGrid/StatsCard=2. Component name is read from Tambo's `content.name` (SDK field), NOT `content.componentName`. Maps are forced to minimum `panelHeight()` even when saved layouts exist. `isCompactComponent()` identifies StatsGrid/StatsCard/InsightCard/DatasetCard/QueryDisplay/DataCard. These get `h-auto` on touch. **Edit with AI**: Pencil button on interactable panels (GeoMap, Graph, DataTable, TimeSlider, ObjexViewer). Uses `useTamboInteractable().setInteractableSelected()` to mark the component for AI focus. One-shot selection, auto-cleared when AI finishes responding.

**Data service (modular)**:
- `src/services/datasets/` - 9 dataset modules + registry index
- `src/services/cross-indices/` - 11 cross-index modules + registry index
- `src/services/resolvers.ts` - Weather (GitHub state file at walkthru-weather-index repo, single fetch, fallback to HEAD probing) and Overture release resolution (GitHub state file at walkthru-overture-index repo, fallback `2026-03-18.0`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [walkthru-earth/ai](https://github.com/walkthru-earth/ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
