---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Situation Monitor is a real-time situation monitoring dashboard. The codebase
is **multi-region**: the backend schedules and fetches **all** regions in
`regions/index.ts` `allRegions` concurrently, broadcasts every region's data
to every SSE client, and the frontend picks which region to surface (header
switcher, persisted in localStorage). The `REGION` / `PUBLIC_REGION` env vars
only set the *default/initial* selection. Current regions:

- **`dc`** — Washington, DC + DMV (PulsePoint EMS1205, MD CHART, MDOT WZDx,
  VDOT NoVA feeds, DC Open Data, DC 311, WMATA, AlertDC, ShotSpotter,
  MoCo/PG crime, OpenMHz DC Fire & EMS scanner audio)
- **`boise`** — Boise / Ada County / Treasure Valley (PulsePoint EMS1169
  ACCESS, BPD Crimes ArcGIS, Ada County CrimeMapper, ITD WZDx + ACHD RITA
  roadwork, Idaho 511 live incidents + Waze reports, Idaho 511 cameras,
  NWS Boise webcams)

Shared (region-agnostic) fetchers: NWS weather alerts, Open-Meteo current
conditions, AirNow, OpenSky aircraft, news RSS, NIFC WFIGS wildfires, USGS
earthquakes, NWS river gauges. Each is instantiated
per-region by the pack file with the region's config (zones, timezone, bbox,
feeds) passed to its constructor — no fetcher reads a global region.

## Region Pack Architecture

Each region exports a `RegionPack` from `regions/<id>.ts` (see
`regions/types.ts` for the contract). A pack bundles:
- **Static config**: `city`/`state`/`timezone`, `defaultCenter`, `openSkyBounds`, `nwsZones`, `sourcesWithCompleteListing`
- **Fetcher arrays** grouped by scheduling profile: `cameraFetchers`, `trafficIncidentFetchers`, `crimeFetchers`, `shotspotterFetchers`, `emergencyAlertFetchers`
- **Singleton-or-null fetchers**: `pulsePointFetcher`, `transitFetcher`, `scannerFetcher`, `twitterFetcher`
- **News config**: RSS feeds + region-specific area keywords / location regex patterns

`regions/index.ts` exports `allRegions` (what the aggregator iterates —
it has zero hardcoded region knowledge), `regionsById`, and
`defaultRegionId` (from `process.env.REGION`, a frontend-default hint only).

Incident lifecycle notes:
- Sources listed in `sourcesWithCompleteListing` are snapshot feeds: absence
  from a successful poll implies cleared (including empty snapshots — their
  fetchers declare `incidentSource` on `BaseFetcher` so this works when a
  feed empties). They are exempt from the age-based cleanup sweep.
- Other sources age out per `expirationMs` in `aggregator.cleanupStaleIncidents`;
  a fetcher's own fetch-window filter must stay inside that window or records
  will clear/re-add in a loop (see bpd-crime / dc-shotspotter for the pattern).
- Normalizers should derive `updatedAt` from feed fields, never `now` — the
  aggregator diffs on `updatedAt` to decide what to re-broadcast.
- Geocoding is region-scoped: `geocache.geocode(address, {city, state, center})`.

## Key Limitations & Solutions

| Limitation | Solution |
|------------|----------|
| DC Metro Police radios encrypted since 2011 | No police scanner data available |
| Boise PD on SWIRC P25 Phase II (some channels encrypted) | Same — link out to RadioReference / PulsePoint instead |
| Neither city publishes raw Fire/EMS CAD | **PulsePoint scraping** via Playwright (DC: EMS1205, Boise: EMS1169 / ACCESS) |
| Twitter API costs $100/month | Optional `TWITTER_BEARER_TOKEN` (DC `@dcfireems` only) |
| Boise crime data lags ~1 month | bpd-crime uses a 60-day fetch window + matching 60-day expiry; near-real-time `BPD_CallsForService` exists but not yet wired |
| DC ShotSpotter feed appears stale (no new detections since ~2026-04) | 30-day fetch window — the layer is honestly empty until the feed revives |
| ITD camera API requires a key | Skipped — Boise's `cameraFetchers` only ships curated NWS Boise airport cams |
| OpenMHz blocks Node's TLS fingerprint (403) while serving curl/browsers | Scanner fetcher shells out to the system curl; DC Fire & EMS call audio plays in the scanner panel. No Idaho systems exist on OpenMHz, so Boise keeps link-outs |

## Build & Development Commands

This is a Turborepo + npm workspaces monorepo. Workspace package names: `@situation-monitor/frontend`, `@situation-monitor/backend`.

**Windows shortcut:** `start-monitor.bat` at the repo root (personal,
gitignored — exists only on the owner's machine) does `docker-compose up -d`
+ `npm run dev`, and runs `docker-compose down` on exit.

**Note on `npm run clean`:** the per-workspace `clean` scripts use POSIX `rm -rf` and will fail in plain PowerShell — use Git Bash / WSL or delete `dist/`, `.svelte-kit/`, `build/`, `node_modules/` manually.

```bash
# From repo root
npm install              # Install all dependencies
npx playwright install chromium   # One-time: browser for PulsePoint scraping
npm run dev              # Start frontend + backend with hot-reload (via turbo)
npm run docker:up        # Start Redis (OPTIONAL — falls back to in-memory cache)
npm run docker:down      # Stop Redis
npm run build            # Production build (both workspaces)
npm run test             # Run tests once and exit (both workspaces)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ripleyhhunter/Situation-Monitor](https://github.com/ripleyhhunter/Situation-Monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
