---
trigger: always_on
description: A **72-inch wall display** dashboard for the Bangkok Governor's Integrated Operations Center. Shows real-time urban data: traffic, flooding, air quality, citizen complaints, news, satellite imagery, CCTV feeds, and market signals. Designed to make a Governor say "this is the best command center I've ever seen."
---

# Bangkok Governor's IOC — Smart City Thailand Monitor

## What This Project Is

A **72-inch wall display** dashboard for the Bangkok Governor's Integrated Operations Center. Shows real-time urban data: traffic, flooding, air quality, citizen complaints, news, satellite imagery, CCTV feeds, and market signals. Designed to make a Governor say "this is the best command center I've ever seen."

This is NOT a generic smart city template. It is a production-grade, data-dense, dark-themed operational dashboard that pulls from 30+ live data sources and renders everything on a single screen with zero wasted pixels.

## Architecture

### Monorepo Structure
```
apps/
  web/          # React 18 + Vite 5 + Leaflet (frontend)
  api/          # Fastify 5 (backend, 25+ adapters, sync engine)
packages/
  shared/       # TypeScript types, seed data, map layers
```

### Tech Stack
- **Frontend:** React 18, Vite 5, Leaflet, TanStack Query
- **Backend:** Fastify 5, TypeScript
- **Maps:** Leaflet + Mapbox/ESRI/CartoDB tiles, Overpass API for road GeoJSON
- **Data:** 29 adapters pulling from CKAN portals, RSS feeds, REST APIs, STAC catalogs
- **No database** — all state is in-memory with periodic snapshot to `tmp/api-state.json`

### Deployment
| Service | Platform | URL |
|---------|----------|-----|
| Frontend (static) | Cloudflare Pages | `bangkok-ioc.pages.dev` |
| API | Render | `smart-city-monitor-api.onrender.com` |

- **Cloudflare Pages** builds from `main` branch using `netlify.toml`
- **Render** builds from `main` branch using `render.yaml`
- Push to `main` = auto-deploy both services
- Working branch: `codex/award-alignment`

### Build Commands
```bash
npm run build -w packages/shared    # Must build first (shared types)
npm run build -w apps/api           # Backend
npm run build -w apps/web           # Frontend (includes tsc check)
```

### Dev Servers
```bash
npm run dev -w apps/api             # localhost:4000
npm run dev -w apps/web             # localhost:5173
```

Frontend uses `VITE_API_BASE_URL` env var. Locally it defaults to `""` (same-origin proxy or direct). In production it points to the Render API.

## Design Philosophy — CRITICAL

Read the parent `~/Projects/CLAUDE.md` for the full manifesto. Here are the rules that matter most for this project:

### Visual Rules (HARD REQUIREMENTS)
- **border-radius: 0px** everywhere. No rounded corners. The only exceptions: small status dots (50% for circles), input fields (functional 2px max).
- **No backdrop-filter blur.** No glassmorphism. Flat, honest panels.
- **No hover animations.** No bouncy transitions. Subtle opacity changes only.
- **No default blue** (#3B82F6). Use project palette: dark bg `rgb(10, 14, 20)`, text `rgb(232, 237, 243)`, accent colors from data meaning (red=danger, green=safe, amber=warning, cyan=data).
- **No boxed SaaS grid layouts.** Asymmetric, editorial, information-dense.
- **Dark theme.** This is an operations center, not a marketing site.

### Information Density
- Every pixel must convey meaning. No decorative spacing.
- The sidebar should pack 12-14 sections into ~800px without scrolling.
- CSS gaps: 0.2-0.3rem between sections. Padding: 0.25-0.4rem inside cards.
- Compress aggressively: single-line items, inline pills, 2-column micro-grids.
- Show MORE data in LESS space. If a section has 3 items and the data has 8, show 8 in a tighter format.

### Typography
- Body: Inter. Headings: Manrope. Thai: Noto Sans Thai.
- Weight contrast over size contrast. Use font-weight 300 for atmosphere, 600 for emphasis.
- Eyebrow labels: 0.6rem, uppercase, letter-spacing 0.04em, opacity 0.6.

## Data Architecture

### Adapter Pattern
Every external data source has an adapter in `apps/api/src/adapters/`. Pattern:

```typescript
export async function syncSourceName(): Promise<AdapterSyncResult> {
  const payload = await fetchJsonOrNull<ResponseType>(config.endpoint);
  if (!payload) return buildResult({ sourceId, status: "stale", ... });
  // Transform to normalized types
  return buildResult({ sourceId, status: "live", newsItems, projectRecords, ... });
}
```

All adapters return `AdapterSyncResult` which can include:
- `newsItems: NewsItem[]` — syndicated content
- `projectRecords: ProjectRecord[]` — tracked initiatives
- `mapFeatureCollections: MapFeatureCollection[]` — GeoJSON for the map
- `mediaFeeds: MediaFeedItem[]` — video/stream links
- Various snapshot patches: `resiliencePatch`, `traffyFonduePatch`, `floodStatusPatch`, `trafficCongestionPatch`, `socialListeningPatch`, `marketSnapshotPatch`

### Sync Engine (`apps/api/src/services/sync.ts`)
- **Ops sync** (every 60s): traffic, air quality, weather, flood, Traffy Fondue
- **Full sync** (every 180s): all of the above + news, catalogs, satellites, markets, CKAN portals

### CKAN Adapters
Three Thai government CKAN portals share helpers in `ckanHelpers.ts`:
- **BMA City Data Portal** (citydataportal.bangkok.go.th) — Bangkok municipal data
- **TAT Tourism** (datacatalog.tat.or.th) — visitor statistics

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nonarkara/smart-city-thailand-monitor](https://github.com/Nonarkara/smart-city-thailand-monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
