---
trigger: always_on
description: This file provides guidance to AI coding agents working in this repository. `CLAUDE.md` is a symlink
---

# AGENTS.md

This file provides guidance to AI coding agents working in this repository. `CLAUDE.md` is a symlink
to this file, so every harness reads the same instructions.

## Repository Overview

This is the GenAI Prices project - a database and tools for calculating LLM inference API pricing. The project includes:

- **Price Data**: YAML files in `prices/providers/` with pricing information for 30+ LLM providers
- **Unit Registry**: `prices/units.yml` defines every billable unit and its price key
- **Packages**: `packages/python/` (PyPI `genai-prices`) and `packages/js/` (npm `@pydantic/genai-prices`) —
  two implementations that must stay behaviourally identical
- **Data Pipeline**: Tools to build JSON schemas, validate data, and update from external sources
- **Price Sources**: Integration with Helicone, OpenRouter, LiteLLM, and other pricing sources

## Architecture

### Core Components

1. **Price Data Sources** (`prices/providers/*.yml`): YAML files containing model pricing information for each provider
2. **Unit Registry** (`prices/units.yml`): the vocabulary of billable units — every valid price key and
   extractor destination is derived from it
3. **Data Pipeline** (`prices/src/prices/`): Python modules that build, validate, and process pricing data
4. **Packages** (`packages/python/`, `packages/js/`): published libraries for end users to calculate costs
5. **External Data Integration**: Tools to pull and compare prices from external sources

### Key Directories

- `prices/`: Core pricing data and build tools
  - `providers/`: YAML files with provider-specific pricing
  - `units.yml`: the unit registry (see "Adding a unit" below before editing)
  - `src/prices/`: Python package for data processing
  - `new_data/v2/`: the **live** published data — `data.json`, `data_slim.json` and their schemas (generated)
  - `data.json`, `data_slim.json` + schemas: **frozen v1** compatibility snapshots (see "Pricing Data")
- `packages/python/`, `packages/js/`: published packages. `data.py`/`data.ts` and `data_units.py`/`dataUnits.ts`
  are generated — never hand-edit them
- `tests/`: Python test suite; JS tests live in `packages/js/src/__tests__/`
- `specs/data-driven-unit-registry/`: authoritative design docs for the unit registry and the v1/v2/v3
  contract rules. Read these before changing anything about units or published artifacts
- `scratch/`: Development/testing files IGNORE THESE FILES

## Development Commands

### Setup

```bash
make install      # Install dependencies and pre-commit hooks
make sync         # Update local packages and uv.lock
```

### Core Development

```bash
make format       # Format code with ruff
make lint         # Check code style and linting
make typecheck    # Run static type checking with basedpyright
make test         # Run the Python tests with coverage (does NOT run the JS suite)
make testcov      # Run tests and generate HTML coverage report
npm run ci        # Build and test the JS package
```

`make all` covers both Python and JavaScript. Run `npm run ci` directly to verify only JavaScript.

### Building and Data Processing

```bash
make build        # build-prices + package-data + inject-providers — use this one
make build-prices # Validate providers and write prices/new_data/v2/* + prices/providers/.schema.json
make package-data # Regenerate the bundled data in packages/python/ and packages/js/
```

Always run `make build`, not `make build-prices` alone. The installed packages read their **bundled**
data (`packages/python/genai_prices/data.py`, `packages/js/src/data.ts`), which only `package-data`
regenerates — so a `calc_price` check run after `build-prices` verifies stale data.

### Price Data Management

```bash
make get-all-prices                    # Download prices from all external sources
make helicone-get                      # Get Helicone prices
make openrouter-get                    # Get OpenRouter prices
make litellm-get                       # Get LiteLLM prices
make simonw-prices-get                 # Get Simon Willison's prices
make huggingface-get                   # Get HuggingFace prices
make ovhcloud-get                      # Get OVHcloud AI Endpoints prices
make quicksilverpro-get                # Get QuickSilver Pro prices
make get-update-price-discrepancies    # Download and update price discrepancies
make check-for-price-discrepancies     # Check for price discrepancies
make detect-deprecated                 # Detect models that may be deprecated or removed
make collapse-models                   # Collapse duplicate similar models
```

`make help` lists every target. These importers run against live third-party APIs and nothing in CI
exercises them, so they break silently when an upstream schema changes — if one returns suspiciously
little, suspect the importer before the data.

## Important Notes

### Pricing Data

- **v2 is the live feed.** `prices/new_data/v2/data.json` is what the published packages auto-update
  from, and it goes live on merge to `main` — independent of any package release.
- **v1 is frozen.** `prices/data.json`, `prices/data_slim.json` and their two schemas are compatibility

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pydantic/genai-prices](https://github.com/pydantic/genai-prices) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
