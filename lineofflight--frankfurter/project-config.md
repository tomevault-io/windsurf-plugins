---
trigger: always_on
description: Frankfurter is a free and open-source currency data API built with Ruby that tracks reference exchange rates from 50+ institutional sources (central banks, the IMF, the Federal Reserve, etc.).
---

# Frankfurter

Frankfurter is a free and open-source currency data API built with Ruby that tracks reference exchange rates from 50+ institutional sources (central banks, the IMF, the Federal Reserve, etc.).

## Architecture

- Roda
- SQLite with Sequel
- Puma
- Rufus scheduler
- Foreman
- Cloudflare CDN

## Project Structure

```
lib/
├── app.rb                       # Main Roda app — mounts v1 and v2
├── base_conversion.rb           # Rebases rates from any base to a common base
├── blender.rb                   # Blends multi-provider rates: rebase → consensus → weighted average
├── bucket.rb                    # Shared SQL bucket expressions for weekly/monthly aggregation
├── cache.rb                     # Cloudflare cache purge
├── carry_forward.rb             # Carries forward most recent provider rate within a lookback window
├── consensus.rb                 # Cross-provider outlier detection (MAD-based)
├── currency.rb                  # Currency model (materialized from rates)
├── currency_coverage.rb         # CurrencyCoverage model (provider-currency join)
├── defunct_currency.rb          # Defunct-currency registry: terminal dates for retired/redenominated ISO codes
├── db.rb                        # Database configuration
├── currency_patches.rb          # Patches Money::Currency: registers historical codes, fixes mangled names
├── log.rb                       # Shared logger
├── monthly_rate.rb              # MonthlyRate model on monthly_rates rollup table
├── no_store_on_error.rb         # Rack middleware: stops CDNs/caches from holding error responses
├── peg.rb                       # Currency peg definitions (from db/seeds/pegs/*.json)
├── peg_anchor.rb                # Peg-aware post-processing: substitutes peg rates, synthesises uncovered pegged quotes
├── provider.rb                  # Provider model: identity, backfill
├── provider/
│   ├── adapters/
│   │   ├── adapter.rb           # Abstract adapter: fetch interface, chunked iteration
│   │   └── <key>.rb             # One adapter per provider (auto-discovered)
│   └── adapters.rb              # Auto-requires all adapters
├── rate.rb                      # Rate model on rates table
├── rate_scopes.rb               # Shared dataset scopes for rate tables (rates, weekly, monthly)
├── rate_validation.rb           # Ingest-validation policy: drops invalid rows on ingest, purges stored ones
├── roundable.rb                 # Currency-aware decimal rounding
├── weekly_rate.rb               # WeeklyRate model on weekly_rates rollup table
├── weighted_average.rb          # Recency-weighted averaging with exponential decay
├── versions/
│   ├── v1.rb                    # Legacy API (ECB-only, frozen)
│   ├── v1/                      # V1 internals (quotes, query, currency names)
│   ├── v2.rb                    # Multi-provider API
│   └── v2/
│       └── rate_query.rb        # V2 rate query builder (blending, filtering)
├── public/
│   ├── favicon.ico              # Served as a static file
│   ├── robots.txt               # Served as a static file
│   ├── v1/openapi.json          # V1 OpenAPI spec
│   └── v2/openapi.json          # V2 OpenAPI spec
└── tasks/
    ├── consensus.rake           # Consensus scan across providers
    ├── db.rake                  # Database migrations and setup
    ├── default.rake             # Default task (lint + test)
    ├── providers.rake           # Dynamic backfill task for all providers
    ├── rollups.rake             # Rebuild weekly/monthly rollup tables
    ├── rubocop.rake             # Linter task
    └── test.rake                # Test suite task

spec/                            # Minitest test suite
db/migrate/                      # Sequel migrations
db/seeds/
    ├── currency_patches.json    # Money::Currency patches (historical codes, name fixes)
    ├── defunct_currencies.json  # Defunct currencies: terminal dates for retired/redenominated ISO codes
    ├── pegs/                    # One JSON file per peg (e.g. aed.json, bam.json)
    └── providers/               # One JSON file per provider (e.g. ecb.json, boi.json)
```

## Key Components

### Adapters (lib/provider/adapters/)
- `Provider::Adapters::Adapter`: Abstract base class — `fetch` interface, `fetch_each` for chunked iteration, sleep no-op in test env
- Adapters are pure data extraction: they know how to talk to an external API and parse its response
- No identity — adapters have no `key` or `name`. Provider model owns identity.
- Optional class methods: `def backfill_range = N`, `def api_key = ENV[...] || raise(ApiKeyMissing)`
- Auto-discovered from `lib/provider/adapters/` via loader

### Models
- `Rate`: Sequel model on `rates` table. Scopes via `RateScopes`: `latest(date)`, `between(interval)`, `only(*quotes)`, `downsample(precision)`
- `WeeklyRate`, `MonthlyRate`: Rollup models on `weekly_rates` / `monthly_rates`, share scopes via `RateScopes`
- `Currency`: Sequel model on `currencies` table. Materialized from rates during backfill. Tracks global date ranges per currency.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lineofflight/frankfurter](https://github.com/lineofflight/frankfurter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
