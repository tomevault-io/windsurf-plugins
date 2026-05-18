---
trigger: always_on
description: **Before marking any feature complete, you MUST update:**
---

# AGENTS.md

## Required: Update Documentation

**Before marking any feature complete, you MUST update:**

1. **`TODO.md`** - Mark tasks complete, add new tasks discovered during implementation
2. **`SPEC.md`** - Update if you changed:
   - UI behavior (how users interact with the app)
   - API endpoints (new, modified, or removed)
   - Data models or database schema
   - Frontend components or layout

This is not optional. The feature is not done until docs are updated.

## Task Management

Feature development and tasks are tracked in `TODO.md`. Check this file for pending work and update it as features are completed.

## Project Overview

Farm Search is a web application for discovering rural and farm properties for sale in NSW, Australia. It displays properties on an interactive map with filtering capabilities based on price, property type, land size, and distance from key locations.

## Tech Stack

- **Backend**: Go 1.24+ with Chi router and sqlx for SQLite
- **Frontend**: Vanilla JavaScript with MapLibre GL JS
- **Database**: SQLite (using modernc.org/sqlite pure Go driver)
- **Build**: Make, Air (live reload)
- **Deployment**: systemd service behind Caddy reverse proxy

## Project Structure

```
farm-search/
├── cmd/
│   ├── server/          # Web server entry point
│   ├── scraper/         # Property scraper CLI
│   └── tools/           # Utility commands (seed, isochrones, distances)
├── internal/
│   ├── api/             # HTTP handlers, routes, middleware
│   ├── db/              # Database connection, queries, schema
│   ├── geo/             # Geographic calculations, isochrones, schools data
│   ├── models/          # Domain types
│   └── scraper/         # Property scrapers (FarmProperty, FarmBuy, REA, Domain), geocoder, browser
├── web/
│   ├── static/          # CSS, JS, and data files
│   └── templates/       # HTML templates
├── scripts/             # Shell scripts and SQL seeds
└── data/                # SQLite database (gitignored)
```

## Key Commands

```bash
make setup        # Install deps and seed sample data
make run          # Start server with live reload (air)
make build        # Build production binaries
make scrape       # Run property scraper
make seed         # Seed sample data
make deploy       # Build and deploy to production
make setup-server # Initial server setup (run once)

# Cadastral data
go run cmd/tools/main.go cadastral        # Fetch lots for properties missing data
go run cmd/tools/main.go cadastral -all   # Re-fetch lots for all properties

# Planning overlays (zoning, bushfire, flood from NSW ePlanning Portal)
go run cmd/tools/main.go overlays         # Fetch overlays for properties missing data
go run cmd/tools/main.go overlays -all    # Re-fetch overlays for all properties

# REA details scraper (fetches full listing details for REA properties)
go run cmd/tools/main.go readetails -scrapingbee $SCRAPINGBEE_API_KEY
go run cmd/tools/main.go readetails -scrapingbee $SCRAPINGBEE_API_KEY -limit 10  # Limit to 10 properties

# Town data (from OpenStreetMap via Overpass API)
go run cmd/tools/main.go extracttowns   # Extract NSW towns/villages/cities to web/static/data/nsw-towns.json
make towns-all                          # Recalculate nearest towns for all properties
make towndrivetimes-all                 # Recalculate drive times to nearest towns

# Hospital data (from OpenStreetMap via Overpass API)
go run cmd/tools/main.go extracthospitals  # Extract NSW hospitals to web/static/data/nsw-hospitals.json
go run cmd/tools/main.go hospitals         # Calculate nearest hospitals (emergency) for all properties
go run cmd/tools/main.go hospitals -all    # Recalculate for all properties
go run cmd/tools/main.go hospitaldrivetimes  # Calculate drive times to nearest hospitals
```

## Development Guidelines

### Go Code

- Use Chi for routing, sqlx for database access
- Place HTTP handlers in `internal/api/handlers.go`
- Database queries go in `internal/db/properties.go`
- Domain models in `internal/models/`
- Use `sql.Null*` types for nullable database fields

### Frontend

- Vanilla JS only, no frameworks or build steps
- MapLibre GL JS for mapping
- Keep JS modular: `api.js`, `map.js`, `filters.js`, `app.js`
- CSS in single `style.css` file

### Database

- Schema defined in `internal/db/schema.sql`
- Migrations run automatically via `db.New()`
- Use `ON CONFLICT` for upserts

### Testing the API

```bash
curl http://localhost:8080/api/properties
curl http://localhost:8080/api/properties/1
curl http://localhost:8080/api/filters/options
```

## External Services

- **Nominatim**: Geocoding (free, rate-limited to 1 req/sec)
- **Valhalla**: Isochrone generation (local Docker instance or public OSM server)
- **NSW Spatial Services**: Cadastral lot boundaries via ArcGIS REST API
- **FarmProperty.com.au**: Primary property listing source (no bot protection)
- **FarmBuy.com**: Secondary property listing source (implemented, no bot protection)
- **realestate.com.au**: Uses ScrapingBee to bypass Kasada bot protection
- **Domain.com.au (API)**: Uses official API (requires API key from developer.domain.com.au)
- **Domain.com.au (Web)**: Traditional web scraping (no API key required, extracts __NEXT_DATA__)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dstrek/farm-search](https://github.com/dstrek/farm-search) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
