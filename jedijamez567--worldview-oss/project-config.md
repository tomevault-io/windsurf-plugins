---
trigger: always_on
description: - MUST have `postcss.config.mjs` with `@tailwindcss/postcss` plugin — without it, `@import "tailwindcss"` produces ZERO utility classes (no error, just silently broken)
---

# WorldView OSS

## Critical: Tailwind CSS v4 + Next.js 16 Turbopack
- MUST have `postcss.config.mjs` with `@tailwindcss/postcss` plugin — without it, `@import "tailwindcss"` produces ZERO utility classes (no error, just silently broken)
- MUST pin `tailwindcss@4.0.7` and `@tailwindcss/postcss@4.0.7` — v4.1.18+ crashes Turbopack
- Next.js 16 is Turbopack-only (no `--no-turbopack` flag exists)
- If Tailwind classes aren't working, check postcss.config.mjs first

## Architecture
- 3-column flexbox layout: left panel (220px) | circular globe viewport | right panel (280px)
- Center column is `flex-col`: globe viewport on top, TimelineSlider below
- CesiumJS globe loaded via `next/dynamic` with `ssr: false` — Cesium cannot run server-side
- State: Zustand store at `stores/worldview-store.ts`
- **9 data layers**: flights, satellites, disasters, asteroids, weather, cameras, livestreams, news, militaryActions
  - 3 expandable with sub-filters: flights (Regular/ISS), disasters (6 types), militaryActions (5 types)
- View modes (EO/FLIR/CRT/NV): CSS filters in `components/effects/ViewModeFilter.tsx`
- SVG filter `<defs>` (e.g. FLIR) must be rendered outside `overflow:hidden` containers — use `FlirFilterDefs` export at page level
- Globe is clipped to circle via `.scope-viewport { border-radius: 50%; overflow: hidden }` in globals.css

## Timeline / Historical Data System
- **Timeline slider** beneath the globe viewport: `components/hud/TimelineSlider.tsx` + `CalendarPicker.tsx`
- **Simulation state** in Zustand: `simulationDate`, `simulationHour`, `simulationMinute`, `isLive`
  - Setters (`setSimulationDate`, `setSimulationTime`) sync CesiumJS clock via `Cesium.JulianDate.fromDate()`
  - `resetToLive()` snaps back to current time and resumes CesiumJS clock animation
- **TimelineSlider uses `mounted` state** to suppress hydration mismatch from `Date.now()` drift between SSR and client
- **Data hooks** (`useNews`, `useDisasters`) check `isLive`:
  - Live mode → `/api/news` (graph + RSS + Reddit), `/api/disasters`
  - Historical mode → `/api/news/historical?date=YYYYMMDD&hour=HH`, `/api/disasters?date=YYYYMMDD`
  - Both hooks use `AbortController` to cancel in-flight requests on cleanup
- **Flights, cameras, livestreams, militaryActions** are always live (no historical data available)

## JanusGraph + Cassandra (Knowledge Graph)
- `docker-compose.yml` — Cassandra 4.1.7 + JanusGraph 1.0.0 (ports configurable via env vars)
  - JanusGraph 1.0.0 requires `JANUS_PROPS_TEMPLATE=cql` (NOT `cassandra`)
- `lib/graph/client.ts` — Gremlin client with **3s connection timeout** and **1min backoff** on failure
  - Uses `mimeType: "application/vnd.gremlin-v3.0+json"` (GraphSON) — GraphBinary fails with JanusGraph custom types
  - Uses `gremlin.process.traversal().with_(conn)` (v3.8 API) — NOT `graph.traversal().withRemote(conn)` (v3.6)
  - If JanusGraph isn't running, graph queries fail fast and don't retry for 60s — app still works with RSS/Reddit only
- `lib/graph/schema.ts` — Graph schema: article, person, organization, location, theme vertices + relationship edges
- `scripts/init-graph-schema.ts` — Idempotent schema init (`npm run init-schema`)
- `lib/graph/queries.ts` — Query functions: `getArticlesByDateRange`, `getArticlesByLocation`, `getPersonNetwork`, `getThemeTrends`
- `types/gremlin.d.ts` — TypeScript declarations for the `gremlin` package (no `@types/gremlin` available)
  - `Graph` lives in `gremlin.structure`, NOT `gremlin.driver`

## GDELT GKG Ingestion Pipeline
- `lib/graph/gkg-parser.ts` — Parses GDELT GKG 2.0 27-column TSV files (handles `\r\n` line endings)
- `lib/graph/gkg-downloader.ts` — Downloads + extracts ZIP archives via `adm-zip` (NOT gzip — GDELT uses standard ZIP format)
- `lib/graph/gkg-loader.ts` — Batch upserts into JanusGraph with get-or-create (upsert) pattern
- `scripts/ingest-gkg.ts` — CLI: `--daily`, `--latest`, `--backfill --from YYYYMMDD --to YYYYMMDD`
- GDELT URLs use HTTPS (configured in `lib/constants.ts`)
- Cache/cursor paths configurable via `GKG_CACHE_DIR` and `GKG_CURSOR_PATH` env vars

## News Data Flow (NO external GDELT Doc API)
- `/api/news` — Primary endpoint: JanusGraph (last 4h) + RSS + Reddit. **Does NOT call GDELT Doc API** (removed due to chronic timeouts)
- `/api/news/realtime` — Same sources as `/api/news`, plus DDG as supplemental (often rate-limited)
- `/api/news/historical` — JanusGraph only, queries ±30min window around requested time
- `lib/api/duckduckgo.ts` — DDG results have `NaN` lat/lon (no geo data); callers must filter accordingly
- RSS feeds: BBC Middle East, BBC World, Al Jazeera, Jerusalem Post, France 24 ME, PressTV Iran
  - Tehran Times removed (307 redirect loop), replaced with PressTV
  - Arab News removed (Cloudflare blocked), replaced with France 24 Middle East

## Military Actions Layer
- `/api/military` — GDELT GEO API with `theme:MILITARY` + conflict keyword queries
- `hooks/useMilitary.ts` — Polls every 5 min, stores in Zustand `militaryActions`
- `components/layers/MilitaryLayer.tsx` — Red/orange dots on globe, click for details
- Sub-filters in sidebar: Airstrikes, Missile Strikes, Ground Ops, Naval Ops, Other

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jedijamez567/worldview_oss](https://github.com/jedijamez567/worldview_oss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
