---
trigger: always_on
description: Open-source database of Kenyan road speed limits and road hazards. Four delivery surfaces: a Go REST API backed by PostgreSQL+PostGIS, a SvelteKit web frontend with Leaflet maps, a Chrome extension that overlays speed zones on Google Maps, and a Kotlin Multiplatform mobile app that includes Android Auto projection (Car App Library) in the same APK. The canonical speed data lives as GeoJSON files in `data/geojson/` — the database is populated from these files. Hazard data (bumps, rumble strips, s
---

# Kenya Speed Limits — CLAUDE.md

## What This Project Is

Open-source database of Kenyan road speed limits and road hazards. Four delivery surfaces: a Go REST API backed by PostgreSQL+PostGIS, a SvelteKit web frontend with Leaflet maps, a Chrome extension that overlays speed zones on Google Maps, and a Kotlin Multiplatform mobile app that includes Android Auto projection (Car App Library) in the same APK. The canonical speed data lives as GeoJSON files in `data/geojson/` — the database is populated from these files. Hazard data (bumps, rumble strips, speed cameras) lives in `data/geojson/hazards/`.

The KMP mobile app lives in a separate sibling repo, [speed-ke-mobile](https://github.com/Arthur-Kamau/speed-ke-mobile), not in this repo. It consumes the same deployed API. `kmp/` and `android-auto/` are gitignored here in case a local checkout is placed alongside this repo for convenience, but they are not tracked or pushed from this repo. (The former standalone `android-auto/` project was merged into the KMP app and deleted.)

## Architecture

```
cmd/api/              → API server entrypoint (gin, port 8080)
cmd/scraper/          → Data scraper / seed loader CLI
internal/api/         → HTTP handlers + router (gin + cors)
internal/db/          → PostgreSQL connection pool (pgx) + spatial queries
internal/models/      → Go structs (RoadSegment, RoadHazard, BBoxQuery, Stats)
internal/scraper/     → Kenya Law scraper (colly) + GeoJSON seed loader
data/geojson/         → Source-of-truth speed limit data (GeoJSON FeatureCollections)
data/geojson/hazards/ → Road hazard data (bumps, rumble strips, speed cameras)
migrations/           → PostgreSQL + PostGIS DDL (golang-migrate format)
frontend/             → SvelteKit 2 + Svelte 5 (runes mode) + Leaflet + TypeScript
extension/            → Chrome Manifest V3 extension
```

Mobile app (separate repo, [speed-ke-mobile](https://github.com/Arthur-Kamau/speed-ke-mobile)):

```
kmp/                  → Kotlin Multiplatform app (Compose, OSMDroid map, Android target)
                        + Android Auto projection (Car App Library 1.4.0 CarAppService)
```

## Commands

### Go backend
```bash
docker compose up -d                    # Start PostgreSQL+PostGIS
migrate -path migrations -database "$DATABASE_URL" up  # Run migrations
go run cmd/scraper/main.go --seed       # Load GeoJSON into database
go run cmd/api/main.go serve            # Start API on :8080
go build ./...                          # Build all packages
go vet ./...                            # Lint
go test ./...                           # Test
make dev                                # All-in-one: db + migrate + seed + serve
```

### Frontend (SvelteKit)
```bash
cd frontend
pnpm install                             # Install deps
pnpm run dev                             # Dev server on :5173 (proxies /api to :8080)
pnpm run build                           # Production build
npx svelte-check --tsconfig ./tsconfig.json  # Type check
```

### Data management
```bash
go run cmd/scraper/main.go --scrape --output data/scraped.json  # Scrape Kenya Law
# Regenerate static fallback after editing GeoJSON:
python3 -c "import json,glob; ... " > frontend/static/speeds.json
```

## Code Conventions

- **Go**: Standard library style. `internal/` for non-exported packages. pgx for Postgres (not database/sql). Gin for HTTP. No ORM.
- **Frontend**: Svelte 5 runes mode (`$state`, `$derived`, `$effect`, `$props`). No Svelte 4 stores or reactive statements. TypeScript strict. Components in `src/lib/components/`, services in `src/lib/services/`, types in `src/lib/types/`.
- **GeoJSON**: Each file is a FeatureCollection. Coordinates are `[longitude, latitude]` (GeoJSON standard). Properties must include: `road_name`, `speed_limit_kmh`, `road_class` (urban|peri_urban|highway|expressway), `direction`, `source`, `verified`, `county`, `last_updated`.
- **road_class** values are constrained by a CHECK in the database: `urban`, `peri_urban`, `highway`, `expressway`.

## Key Design Decisions

- **GeoJSON is source of truth**, not the database. The database is derived via the seed command, which **truncates and fully reloads `road_segments` inside one transaction on every run** (see `internal/scraper/seed.go`) — this is what it means for the DB to be "derived": renamed/edited/removed segments in GeoJSON are reflected exactly, with no stale or duplicate rows left behind from previous deploys. `road_segments` has no other write path, so this is safe. Edit `data/geojson/*.geojson` to change speed data.
- **Frontend works offline** — if the Go API is unreachable, it falls back to `frontend/static/speeds.json` (a bundled copy of all GeoJSON data).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Arthur-Kamau/road-speed-ke](https://github.com/Arthur-Kamau/road-speed-ke) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
