---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

@GUIDELINES.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Property Shared is a FastAPI service + pure-Python core library for UK property data. It integrates:
- **PPD** (Price Paid Data) - Land Registry transactions via SPARQL/Linked Data API
- **EPC** - Energy Performance Certificates (requires API credentials)
- **Rightmove** - Property listings via scraping with built-in politeness
- **Planning** - UK council planning applications via vision-guided browser automation (99 verified councils)
- **Stamp Duty** - SDLT calculator with April 2025 bands, additional property/FTB/non-resident surcharges
- **Block Analyzer** - Groups PPD transactions by building to find bulk-buy opportunities
- **Companies House** - Company search and lookup via free API (requires API key)

## Commands

```bash
# Install dependencies (with dev extras)
uv sync --extra dev

# Run API server
uv run --env-file .env property-api              # production mode
uv run --env-file .env uvicorn app.main:app --reload  # dev mode with reload

# Run CLI (core mode - no server needed)
uv run --extra cli property-cli meta
uv run --extra cli property-cli ppd comps "SW1A 1AA" --months 24
uv run --extra cli property-cli rightmove search-url "SW1A 1AA"
uv run --extra cli property-cli calc stamp-duty 300000
uv run --extra cli property-cli ppd blocks "B1 1AA"
uv run --extra cli property-cli companies search "Tesco"
uv run --extra cli property-cli analysis yield "NG1 1AA"
uv run --extra cli property-cli analysis rental "NG1 1AA"

# CLI targeting running API (add --api-url)
uv run --extra cli property-cli ppd comps "SW1A 1AA" --api-url http://localhost:8000

# Tests
uv run --extra dev --extra apps pytest                        # unit tests (mocked)
RUN_LIVE_TESTS=1 uv run --extra dev --extra apps pytest       # live network tests

# Single test
uv run --extra dev --extra apps pytest tests/test_ppd_service_live.py -v
```

## Fly.io Deployment — Two Apps, One Repo

This repo deploys to **two separate Fly.io apps** with different Dockerfiles and configs:

| App | Fly config | Dockerfile | URL | What it is |
|-----|-----------|-----------|-----|------------|
| `property-shared` | `fly.toml` | `Dockerfile` | property-shared.fly.dev | FastAPI REST API — core data layer, `/v1/health`, 2GB RAM |
| `propertydata` | `fly.app.toml` | `Dockerfile.app` | propertydata.fly.dev | FastMCP MCP app — tools + Prefab dashboards, `/health`, 512MB |

`property-shared` exposes both the REST API (`/v1/`) and a plain MCP server (`/mcp`) — works in any MCP client. `propertydata` is the MCP app with Prefab UI dashboards — claude.ai only.

**Deploy manually:**
```bash
# property-shared (default)
fly deploy --ha=false

# propertydata (must specify config)
fly deploy --config fly.app.toml --ha=false
```

**CI (release.yml)** — runs both on `release: published`. Requires two GitHub secrets:
- `FLY_API_TOKEN` — scoped to `property-shared` (`fly tokens create deploy -a property-shared`)
- `FLY_API_TOKEN_PROPERTYDATA` — scoped to `propertydata` (`fly tokens create deploy -a propertydata`)

## Architecture

```
property_core/              # Pure Python library (no FastAPI, no DB assumptions)
├── models/                 # Domain Pydantic models
│   ├── ppd.py              # PPDTransaction, PPDCompsResponse, SubjectProperty, etc.
│   ├── epc.py              # EPCData
│   ├── rightmove.py        # RightmoveListing, RightmoveListingDetail
│   ├── report.py           # PropertyReport, YieldAnalysis, RentalAnalysis, etc.
│   ├── block.py            # BlockUnit, BlockBuilding, BlockAnalysisResponse
│   └── companies_house.py  # CompanyRecord, CompanySearchResult, CompanyOfficer
├── ppd_client.py           # Transport: Land Registry SPARQL + Linked Data API → typed models
├── epc_client.py           # Transport: EPC registry (async) → typed EPCData models
├── rightmove_scraper.py    # Transport: listings scraper (sync) → typed Pydantic models
├── rightmove_location.py   # Transport: search URL builder (sync)
├── postcode_client.py      # Transport: postcodes.io → typed PostcodeResult model
├── companies_house_client.py # Transport: Companies House API (sync httpx, basic auth)
├── ppd_service.py          # Domain service: PPD comps, search, stats (sync)
├── planning_service.py     # Domain service: council matching + URL building (sync)
├── report_service.py       # Orchestrator: multi-source aggregation (async)
├── rental_service.py       # Rental analysis with optional yield calculation (async)
├── yield_service.py        # Yield analysis: PPD sales + Rightmove rentals → YieldAnalysis
├── interpret.py            # Opt-in interpretation helpers (classify_yield, generate_insights, etc.)
├── enrichment.py           # EPC enrichment pipeline + compute_enriched_stats()
├── address_matching.py     # Fuzzy address matching for EPC enrichment
├── stamp_duty.py           # SDLT calculator: April 2025 bands, surcharges, FTB relief
├── block_service.py        # Block analyzer: groups PPD transactions by building
├── planning_scraper.py     # Vision-guided planning portal scraper (Playwright + OpenAI)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paulieb89/property-shared](https://github.com/paulieb89/property-shared) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
