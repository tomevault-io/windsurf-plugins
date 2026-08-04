---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Google Maps Business Extractor (`gmaps-extractor`) -- A pip-installable Python library that reverse-engineers Google Maps' internal protobuf API to extract business information at scale using raw HTTP requests. No browser automation, no official API key.

**Input:** Area name (e.g., "New York, USA") + Category (e.g., "lawyers")
**Output:** JSON + CSV files with all matching businesses, including details and reviews.

**Package names:** `gmaps-extractor` (PyPI), `gmaps_extractor` (import)
**Current version:** 2.0.0

## Installation

```bash
# Core library (httpx only, no FastAPI)
pip install gmaps-extractor

# With optional FastAPI server (for CLI or direct API access)
pip install gmaps-extractor[server]

# Development (testing, linting, type checking)
pip install -e ".[dev]"

# Documentation
pip install -e ".[docs]"
```

## Commands

### Running Tests
```bash
pytest                          # Run all 570+ tests
pytest tests/ -x               # Stop on first failure
pytest tests/test_client.py    # Run specific test file
pytest -k "test_search"        # Run tests matching pattern
coverage run -m pytest && coverage report  # With coverage
```

### Linting and Type Checking
```bash
ruff check gmaps_extractor/    # Lint
ruff format gmaps_extractor/   # Format
mypy gmaps_extractor/          # Type check
```

### Building
```bash
python -m build                # Build sdist and wheel
mkdocs build                   # Build documentation site
mkdocs serve                   # Local docs preview at localhost:8000
```

## Library Usage (v2.0.0)

### Sync (default -- no server needed)
```python
from gmaps_extractor import GMapsExtractor

with GMapsExtractor(proxy="http://user:pass@host:port") as extractor:
    result = extractor.collect("New York, USA", "lawyers", enrich=True)
    result = extractor.collect_v2("Paris, France", "restaurants", reviews=True)
```

### Async
```python
async with GMapsExtractor(proxy="http://user:pass@host:port") as extractor:
    result = await extractor.async_collect_v2("NYC", "lawyers", enrich=True)
    async for biz in extractor.stream_collect_v2("NYC", "lawyers"):
        print(biz["name"])
```

### Events
```python
from gmaps_extractor import GMapsExtractor, EventType

def on_found(event):
    print(f"Found: {event.data}")

with GMapsExtractor(proxy="...", on_business_found=on_found) as ext:
    ext.collect("NYC", "lawyers")
```

### Legacy (server-based, requires `[server]` extra)
```python
with GMapsExtractor(proxy="...", use_server=True) as extractor:
    result = extractor.collect("NYC", "lawyers")
```

## Architecture (v2.0.0)

```
gmaps_extractor/
├── __init__.py              # Package entry: exports, lazy imports, config shim, NullHandler
├── extractor.py             # GMapsExtractor class (high-level API) + CollectionResult
├── client.py                # GMapsClient: sync direct HTTP to Google Maps (DEFAULT)
├── async_client.py          # AsyncGMapsClient: async direct HTTP via httpx.AsyncClient
├── settings.py              # GMapsSettings dataclass (replaces monkey-patching config)
├── events.py                # EventEmitter + EventType + Event (lifecycle hooks)
├── progress.py              # ProgressReporter (attaches to EventEmitter)
├── config_manager.py        # ExtractorConfig (DEPRECATED - legacy bridge to config.py)
├── exceptions.py            # Custom exception hierarchy
├── _config_defaults.py      # Safe fallback config for pip-only installs
├── config.py                # Proxy, cookies, rate limits (gitignored, user-specific)
├── config.example.py        # Template config with placeholders
├── server.py                # FastAPI server (OPTIONAL: pip install gmaps-extractor[server])
├── cli.py                   # CLI: gmaps-collect (V1)
├── cli_v2.py                # CLI: gmaps-collect-v2 (V2)
├── cli_enrich.py            # CLI: gmaps-enrich-reviews
├── decoder/
│   ├── pb.py                # Decodes Google's !field_type_value protobuf format
│   ├── curl.py              # Parses curl commands
│   └── request.py           # Combined request decoder
├── parsers/
│   ├── business.py          # Extracts businesses from search response arrays
│   ├── place.py             # Extracts place details (hours, phone, website, photos)
│   └── reviews.py           # Extracts reviews from place responses
├── geo/
│   ├── grid.py              # GridCell, AreaBoundary, grid generation, boundary math
│   └── nominatim.py         # OpenStreetMap Nominatim API (boundaries + sub-areas)
└── extraction/
    ├── search.py            # Builds search queries (uses GMapsClient or legacy server)
    ├── enrichment.py        # Sync enrichment (details + reviews per business)
    ├── collector.py         # V1 sync orchestrator (parallel grid search)
    ├── collector_v2.py      # V2 sync orchestrator (resumable, adaptive, JSONL)
    ├── async_collector.py   # V2 async orchestrator (asyncio.gather/TaskGroup)
    └── async_enrichment.py  # Async enrichment (asyncio.Semaphore concurrency)

tests/                       # 570+ tests (pytest + pytest-asyncio)
docs/                        # MkDocs Material site

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [promisingcoder/GoogleMapsCollector](https://github.com/promisingcoder/GoogleMapsCollector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
