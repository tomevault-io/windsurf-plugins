---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

**cloud-cap-hana-swapi** is a learning-focused SAP Cloud Application Programming Model (CAP) sample using the Star Wars universe to demonstrate production patterns — especially many-to-many relationships — across multiple database backends (SAP HANA, SQLite, PostgreSQL).

All implementation code lives under the `cap/` subdirectory. Run all npm commands from within `cap/`.

## Commands

```bash
cd cap

# Local development
npm run sqlite        # Start with SQLite (no external services needed)
npm run watch         # Hybrid dev against SAP HANA Cloud (requires .cdsrc-private.json)
npm run pg            # PostgreSQL profile watch

# Testing
npm test              # Full test suite (model + handler + data conversion, 60s timeout)
npm run test:handler  # Handler/service layer tests only
npm run test:profile  # Fast regression gate — use before committing
npm run test:migration # Data conversion/loading tests

# Build & Deploy
npm run build         # CDS build + generate typed models (required before HANA deployment)
npm run types         # Regenerate CDS typed models → @cds-models/
npm run build_sqlite  # Deploy schema to SQLite
npm run build_pg      # Deploy schema to PostgreSQL
npm run hana          # Deploy to HANA HDI container ("starwars")

# Data loading
npm run load          # Load Star Wars fixture data (hybrid/HANA profile)
npm run load_sqlite   # Load fixture data into SQLite
npm run load_pg       # Load fixture data into PostgreSQL

# Data scraping
npm run scrape               # full run, cache-first (fast — uses committed cache)
npm run scrape:films         # films-only, no episodes (reproduces original dataset)
npm run scrape:bypass-cache  # fetch fresh from Wookieepedia (requires confirmation)

# Docs generation
npm run openapi       # Generate OpenAPI docs → docs/
npm run asyncapi      # Generate AsyncAPI docs
```

Tests use the Node.js built-in `node:test` runner (not Jest/Mocha). Run a single test file directly:
```bash
node --test cap/test/handler.test.js
```

After any CDS model changes, run `npm run build` in `cap/` to regenerate artifacts.

## Architecture

### Layer Separation

```
cap/db/          Domain model + persistence (*.cds), profile-specific extensions
cap/db/src/      HANA migration tables (.hdbmigrationtable) for schema evolution
cap/db/last-dev/ Last-deployed CSN snapshot (used by HANA schema migration)
cap/srv/         Service layer: contracts (*-service.cds), Fiori annotations (*-fiori.cds),
                 authorization (services-auth.cds), runtime handlers (*.js), server config (server.js)
cap/app/         UI frontends — five apps: film/, people/, show/, media/ (Fiori Elements),
                 viewer/ (custom HTML/JS app)
cap/test/        Automated tests by layer (model, handler, data migration)
cap/docs/        Generated docs (OpenAPI, AsyncAPI) and learning materials
cap/labs/        Hands-on exercises (lab-01 through lab-05)
cap/@cds-models/ Generated CDS typed models (via @cap-js/cds-typer, git-ignored)
```

### Domain Model (`cap/db/schema.cds`)

Core entities: **Film**, **People**, **Planet**, **Species**, **Starship**, **Vehicle**, **Show**, **Episode**. Many-to-many relationships use explicit junction entities (`Film2People`, `Film2Planets`, `Episode2People`, `Episode2Planets`, etc.) with redirected projections in services. All entities use `managed` + `cuid` from `@sap/cds/common`.

`Episode` is a composition child of `Show` — episodes cascade-delete with their parent show. Five `Episode2*` junction tables (`Episode2People`, `Episode2Planets`, `Episode2Starships`, `Episode2Vehicles`, `Episode2Species`) link episodes to the core entity set.

`Show2Planets`, `Show2Starships`, `Show2Vehicles`, and `Show2Species` are CDS `define view` declarations that aggregate over the corresponding `Episode2*` tables rather than being physical tables.

Profile-specific extensions live in `cap/db/hana/`, `cap/db/sqlite/`, `cap/db/postgres/`. Always check these when making cross-profile changes.

### Service Layer (`cap/srv/`)

Nine services total:

- **Six core entity services**: `StarWarsFilm`, `StarWarsPeople`, `StarWarsPlanet`, `StarWarsSpecies`, `StarWarsStarship`, `StarWarsVehicle`
- **`StarWarsShow`**: Full service exposing `Show`, `Episode`, `Media`, `MediaCharacters`, `Show2People`, `Show2Planets`, and related projections. Has handler logic in `show-service.js` (computes virtual `edit_url` on `Media` reads).
- **`StarWarsEpisode`**: Read-only projections of `Episodes` and `Episode2*` junctions (no handler logic). Separate CDS file with its own Fiori annotations.
- **`DataService`** (`/-data`): Entity metadata/introspection service exposing entity names, columns, and types. Handler in `data-service.js`.

Protocols: OData v4 (primary), OData v2 (adapter), GraphQL (`/graphql`), REST.

**Critical file separation rule:**
- Service contracts → `*-service.cds`
- Fiori/UI annotations → `*-fiori.cds` (never mix into service contracts)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SAP-samples/cloud-cap-hana-swapi](https://github.com/SAP-samples/cloud-cap-hana-swapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
