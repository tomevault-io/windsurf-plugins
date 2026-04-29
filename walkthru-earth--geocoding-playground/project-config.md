---
trigger: always_on
description: pnpm dev              # Start Svelte dev server (playground)
---

# Walkthru Earth Geocoding Playground

## Commands
```
pnpm dev              # Start Svelte dev server (playground)
pnpm build            # Build core THEN playground (order matters)
pnpm check            # svelte-check + tsc
pnpm lint             # Biome check
pnpm lint:fix         # Biome check --write
pnpm format           # Biome format --write
pnpm test             # Run core unit tests (Vitest)
pnpm test:coverage    # Unit tests with coverage report
pnpm test:e2e         # Run Playwright e2e tests (builds + serves playground)
```

Core only: `pnpm --filter @walkthru-earth/geocoding-core build`

### Testing

- **Unit tests** (`packages/core/`): Vitest. Run with `pnpm test`. Config at `packages/core/vitest.config.ts`.
- **E2E tests** (`apps/playground/e2e/`): Playwright (Chromium). Run with `pnpm test:e2e`. Config at `apps/playground/playwright.config.ts`. Tests hit live S3 data, so they need network access and can be slow on first run.
- **Data validation** (`pnpm test:validate`): Queries live S3 parquet files via DuckDB CLI to verify parsed addresses exist in the real data. Generates `test-output/validation/parser-validation.md`.
- **Pre-commit hook**: Runs lint-staged + type-check + build. Unit and e2e tests run in CI only.
- **Test output**: All test artifacts go to `test-output/` (gitignored). Unit coverage HTML at `test-output/unit/coverage/`, Playwright report at `test-output/e2e/report/`.

**This project uses pnpm exclusively. Never use npm or npx.** Use `pnpm` and `pnpx` instead. A PreToolUse hook will block npm/npx commands automatically.

## Architecture

pnpm monorepo with two packages:

- `packages/core/` - `@walkthru-earth/geocoding-core`. Framework-agnostic TS library. DuckDB-WASM wrapper, address parsing, autocomplete engine, search/ranking. Built with Vite + vite-plugin-dts.
- `apps/playground/` - Svelte 5 UI. Tailwind 4 + DaisyUI 5 + MapLibre GL. Consumes core as `workspace:*` dependency.

The core package is designed to work with React, Vue, Next.js, or any framework.

## Upstream Pipeline

The data pipeline lives in a separate repo: `walkthru-earth/walkthru-overture-index`.
Key files: `sql/addresses.sql` (DuckDB 1.5 SQL) + `main.py` (orchestration + s5cmd parallel S3 upload).
Pipeline processes 469M Overture Maps addresses into partitioned GeoParquet tiles on S3.

## Data on S3

Base URL: `https://s3.us-west-2.amazonaws.com/us-west-2.opendata.source.coop/walkthru-earth/indices/addresses-index/v4/release=<RELEASE>/`

The app discovers releases from the S3 listing (`getAvailableReleases()`) and defaults to newest. Current newest is `2026-04-15.0`. Never hard-code a release in docs, always substitute the current one when running queries.

Key files (all Hive-partitioned by country):
- `manifest.parquet` - 39 countries (3 KB)
- `tile_index.parquet` - 17,499 tiles (561 KB)
- `city_index/country=XX/data_0.parquet` - per-country cities
- `postcode_index/country=XX/data_0.parquet` - per-country postcodes
- `street_index/country=XX/data_0.parquet` - per-country streets
- `number_index/country=XX/data_0.parquet` - per-country house numbers
- `geocoder/country=XX/h3_res4=YYY/bucket=NN/data_0.parquet` - address tiles, partitioned by h3 res 4 parent and a per-tile bucket (`_` for small tiles, `01..NN` for large ones)

## Key Conventions

- **Build order**: Always build core before playground (`pnpm build` handles this)
- **HTTPS, not s3://**: DuckDB-WASM uses plain HTTPS for all data access (s3:// fails with 416 on large files)
- **h3_index as BIGINT**: Not VARCHAR hex. 46% smaller, enables row-group pushdown
- **Per-country index files**: Not global. Avoids 73 MB footer overhead in WASM
- **NUMBER_FIRST countries**: US, CA, AU, NZ, BR, MX, CL, CO, UY, SG, HK, TW, GB, IE (house number before street)
- **Street-first countries**: NL, DE, FR, IT, AT, CH, ES, PT, BE + Nordic + Baltic + Balkans
- **Parquet optimization**: All files use ZSTD compression, Parquet v2 data pages. Geocoder tiles have bloom filters on `street` for row-group skipping. number_index uses ROW_GROUP_SIZE 2000 + bloom filters on `street_lower` for HTTP range-request pushdown (~150 KB per query vs full file)
- **`$state.raw` for query results**: Use `$state.raw` (not `$state`) for arrays of immutable DuckDB results (`results`, `cities`, `suggestions`, `countries`) to avoid Svelte 5 deep-proxy overhead
- **JS array search over DuckDB for cached data**: City search uses `searchCities()` from core (sub-ms JS) instead of DuckDB SQL round-trips. Street/postcode autocomplete queries DuckDB in-memory tables
- **`getAvailableReleases()` not `availableReleases`**: Release list is exposed via a getter function, not an exported mutable variable
- **libpostal street-type synonyms**: `packages/core/src/dictionaries/` vendors libpostal canonicalizations for 27 languages. `expandStreetVariants(cc, token)` and `expandDirectional(cc, token)` feed `buildStreetPrefixClause`, which emits `(street_lower LIKE 'x ave%' OR street_lower LIKE 'x avenue%' ...)`. `COUNTRY_LANGUAGES` maps each ISO country to its ISO 639 languages (e.g. `JP → ja`, `CH → de,fr,it`). Regenerate via `pnpm tsx scripts/fetch-libpostal-dicts.ts`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [walkthru-earth/geocoding-playground](https://github.com/walkthru-earth/geocoding-playground) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
