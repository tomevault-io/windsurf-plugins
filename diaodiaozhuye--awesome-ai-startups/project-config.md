---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI Product Data — an open-source, Git-native data repository tracking AI startups worldwide. The project is **product-centric**: a company can have multiple AI products, and products are the core data unit.

Three layers: JSON data files, Python scraper/CLI tooling, and a Next.js website with API routes.

GitHub: `diaodiaozhuye/awesome-ai-startups`

## Commands

### Python (scrapers & CLI)

```bash
pip install -e ".[dev]"          # Install with dev dependencies (pytest, ruff, mypy)
aiscrape validate                # Validate all product JSONs against schema
aiscrape init-db                 # Rebuild SQLite database from JSON files
aiscrape init-db --verify        # Rebuild and verify round-trip fidelity
aiscrape generate-stats          # Regenerate data/index.json and data/stats.json
aiscrape show <slug>             # Display a single company's data
aiscrape scrape --source github --dry-run   # Preview scraper output
aiscrape scrape --source all --limit 50     # Run all scrapers
```

```bash
pytest tests/ -v                             # Run all tests
pytest tests/test_normalizer.py -v           # Run a single test file
pytest tests/test_normalizer.py::test_name -v  # Run a single test
pytest --cov=scrapers --cov-report=term-missing  # Coverage report
ruff check scrapers/ tests/                  # Lint Python
mypy scrapers/ --ignore-missing-imports      # Type check
```

### Website (Next.js)

```bash
npm install                      # Install frontend dependencies
npm run dev                      # Dev server with hot reload
npm run build                    # Build (output in .next/)
npm run lint                     # ESLint
```

## Architecture

### Data Pipeline (Scraper -> Disk)

```
Source Scrapers (scrapers/sources/)
  -> ScrapedCompany (frozen dataclass)
  -> Normalizer (scrapers/enrichment/normalizer.py)    -- standardize URLs, names, countries
  -> Deduplicator (scrapers/enrichment/deduplicator.py) -- match via domain, slug, name similarity
  -> Merger (scrapers/enrichment/merger.py)             -- non-destructive merge (never overwrites manual edits)
  -> SchemaValidator (scrapers/validation/)             -- JSON Schema check
  -> Write to data/products/<slug>.json
```

New scrapers extend `BaseScraper` (scrapers/base.py) and register in `scrapers/sources/__init__.py`. The `ScrapedCompany` is a frozen dataclass -- only `name` and `source` are required; the enrichment pipeline handles the rest.

### Generators

`IndexGenerator` and `StatsGenerator` (scrapers/generators/) produce `data/index.json` and `data/stats.json` -- these are auto-generated and should not be manually edited. Run `aiscrape generate-stats` after data changes.

### Website

Next.js App Router (`trailingSlash: true` in next.config.ts). Deployed on Vercel with auto-detection. The Next.js app lives at the repo root; `data/` is a sibling directory accessed at build time and via API routes.

**API routes** under `src/app/api/` provide server-side data access with ISR caching (1 hour):
- `/api/products?page=1&limit=24&category=...&sort=name` — paginated product listing
- `/api/products/[slug]` — single product detail
- `/api/categories` — category list with counts
- `/api/stats` — aggregate statistics
- `/api/search?q=...&category=...&country=...` — server-side Fuse.js search

i18n via URL segments (`/en/`, `/zh/`). Server-side search powered by Fuse.js. Charts via Recharts. The data layer reads from `data/` via `src/lib/data.ts`.

Key layout: `src/app/[locale]/` -- all pages are under the locale dynamic segment. Components are organized under `src/components/` by domain (company, search, analytics, layout, ui).

### SQLite Data Layer

The website uses SQLite as its primary data source, with JSON file fallback:

- `scrapers/db.py` — `ProductDB` class providing CRUD, FTS search, and aggregation queries
- `data/schema/products.sql` — DDL with FTS5 virtual table and sync triggers
- `src/lib/data.ts` — reads SQLite via `better-sqlite3` (lazy singleton), falls back to JSON if DB is missing
- `aiscrape init-db` — rebuilds the database from `data/products/*.json` files

The database (`data/products.db`) is **not committed to git** — it is rebuilt in CI (`validate-pr.yml` runs `aiscrape init-db` before the website build step). Developers should run `aiscrape init-db` locally after pulling data changes.

## Data Schema

Product JSON files live in `data/products/` and must conform to `data/schema/product.schema.json`.

Required fields: `slug`, `name`, `product_url` (URI), `description`, `product_type`, `category` (enum), `status`.

Valid categories are defined in `data/categories.json` (currently 22). Use the canonical list from that file — do not hardcode category IDs.

The slug must match the filename (without `.json`) and follow pattern `^[a-z0-9-]+$`.

## Bilingual / i18n Rules

The website supports English (`/en/`) and Chinese (`/zh/`). Follow these rules strictly:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [diaodiaozhuye/awesome-ai-startups](https://github.com/diaodiaozhuye/awesome-ai-startups) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->
