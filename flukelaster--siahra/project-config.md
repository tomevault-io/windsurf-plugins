---
trigger: always_on
description: **SIAHRA** (Spatial Intelligence Atlas for Hazard & Resilience Analytics) — a 3D per-province map of Thailand that overlays *actually measured* hazard data (ThaiWater/HII, GISTDA, TMD, USGS/EMSC) on Three.js + React (Vite) with a Cloudflare Worker backend (Durable Objects, R2). The overall plan lives in `docs/SIAHRA-implement-plan.md` (research blueprint); the execution order and task list in `docs/roadmap.md`; deploy steps in `docs/deploy.md`.
---

# SIAHRA — guide for agents and contributors

**SIAHRA** (Spatial Intelligence Atlas for Hazard & Resilience Analytics) — a 3D per-province map of Thailand that overlays *actually measured* hazard data (ThaiWater/HII, GISTDA, TMD, USGS/EMSC) on Three.js + React (Vite) with a Cloudflare Worker backend (Durable Objects, R2). The overall plan lives in `docs/SIAHRA-implement-plan.md` (research blueprint); the execution order and task list in `docs/roadmap.md`; deploy steps in `docs/deploy.md`.

## Non-negotiable rules (data honesty)
- Every data layer must declare a `HazardLayerDescriptor` (`packages/shared-types/src/hazard-layer.ts`): observed / static-reference / illustrative / probabilistic — and the UI must always show when the data is from (`fetchedAt`/`observedAt`)
- **Never invent forecast numbers** — no "% chance of flooding" that does not come from a citable model; the "low-lying area" layer is *illustrative*, derived from a DEM, and its legend says so
- Stale data and dead sources must stay visible (dimmed dots, labels, the status bar fed by `/api/v1/health`) instead of silently disappearing; `fetchedAt` is null when a fetch has never succeeded — never render that as "now"
- Historical values (timeline) carry no ThaiWater `situationLevel` → colour is derived from the distance below bank level, and that is stated explicitly
- **Never state a source's condition unless it was actually probed** — a layer that is off has requested nothing, and an unreachable API tells you only that *you could not ask*: neither may be reported as "nothing new was computed" or "the source is quiet". Say which of the two it is; the layer still dims either way
- An artefact that fails the checksum its manifest declares stops feeding the layers derived from it, and the legend says why — but **"not verified" is not "failed"**: a missing checksum suppresses nothing (`docs/dataset.md`)

## Layout
- `apps/web` — **deploy unit 1** (Worker `siahra-web`; `wrangler.jsonc` has **both** `main: worker/index.ts` and an `assets` block — the Worker serves R2 tiles and its own 404/405, and falls through to the asset layer for everything else, which is also what makes `public/_headers` ship and be applied; tile paths come in two shapes — version-addressed `/aoi/{code}/v/{ver}/{layer}/{z}/{x}_{y}.bin` and the legacy `/aoi/{code}/{layer}/...`, both parsed by `worker/tilePath.ts`, which the dev middleware in `vite.config.ts` shares, and a versioned URL never falls back to the legacy object (`docs/dataset.md` §7); measured on prod 2026-08-20, a path the Worker does **not** claim gets the SPA shell as `200 text/html`, never a 404, so a status code alone never proves a file exists — any probe of an `/aoi/` URL has to assert `application/octet-stream` too): React 19 + Vite + Tailwind 4 + three r185 (raw scene graph, not R3F): `src/scene/*` (TerrainTiles LOD, BuildingTiles, FeatureTiles, VegetationTiles, RadarOverlay, floodMask, terrainMaterial shader), `src/components/layout/Map3DCanvas.tsx` assembles every layer, web workers in `src/workers/*`; `og/` builds the link-preview image (`npm run og:image -w apps/web` → `public/og-image.jpg`, 2400×1260, needs the global `playwright-cli`; the map on it is the 77 province outlines from `public/aoi/*/boundary.geojson` — decoration only, no hazard values are drawn) and `index.html` carries the matching `og:*`/`twitter:*` tags with **absolute** `https://siahra-radar.co/...` URLs — after changing the image, bump the filename or re-scrape (Facebook Sharing Debugger / LINE Poker), because scrapers cache by URL
- `apps/api` — **deploy unit 2** (Worker `siahra-api`, no `assets` block any more): `src/router.ts` (route table + rate limit + same-origin guard), DOs in `src/durable-objects/*` (ObservationCacheDO = ThaiWater + history + dams + archive, FloodExtentDO, RadarDO, EarthquakeFeedDO), ingestion in `src/ingestion/*`, permanent archive in `src/archive.ts`
- `apps/etl` — gdal/osmium pipelines: `build:all`, `build:tiles`, `build:building-tiles`, `build:feature-tiles`, `build:landcover-tiles` → small outputs (manifest/overview) land in `apps/web/public/aoi/{code}` (tracked) and the large tiles in `apps/etl/data/tiles` (gitignored, ~5.6 GB, served in dev by middleware in `apps/web/vite.config.ts`; prod = R2); `refresh:manifests` rewrites `manifest.provenance` (per-layer `builtAt`, `publishedAt`, sha256 of `terrain.bin`) and the four tile `urlTemplate`s over artefacts that already exist, without rebuilding a tile — `src/provenance.ts` is the single derivation module both write paths share, and `src/datasetVersion.ts` refuses to reuse a `datasetVersion` whose `builtAt`/`checksums` moved, because that version is a tile URL prefix served `immutable` for a year (`docs/dataset.md`)
- `packages/shared-types` — the data contract between api/web/etl (always change it here first)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flukelaster/SIAHRA](https://github.com/flukelaster/SIAHRA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
