---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Docs

- `docs/architecture.md`: full architectural reference (API clients, GA4 protobuf pattern, batch implementation, retry/quota internals, v0.5 additions)
- `docs/google-setup.md`: GCP project setup, enabling APIs, service account creation, GSC and GA4 permission configuration
- `docs/starter-prompt.md`: ready-to-use audit prompts for Claude Desktop users (full audit, quick check, single-page, reindexing, CrUX, schema)

## Commands

```bash
# Requires Python 3.11+
python3 --version  # must be 3.11 or higher

# Install (dev mode with test deps)
pip install -e ".[dev]"

# Run the MCP server
gsc-mcp
# or
python -m gsc_mcp.server

# Run all tests (339 tests, fully mocked)
pytest tests/ -v

# Run a single test file
pytest tests/test_analytics.py -v

# Run a specific test by name
pytest tests/ -k "test_submit_batch" -v
```

## Architecture

**Entry point**: `src/gsc_mcp/server.py` creates a `FastMCP("gsc-mcp")` instance and registers all 43 tools by iterating `registry.TOOLS`. Dynamic: adding a function to `registry.py` is enough to register it in both the MCP server and the CLI.

**Registry** (`src/gsc_mcp/registry.py`): imports all 43 tool functions and exposes `TOOLS: dict[str, Callable[..., str]]`. An `assert` at import time verifies `set(TOOLS) == set(_ALL_TOOLS)` from `properties.py`, so any mismatch fails loudly at startup.

**CLI** (`src/gsc_mcp/cli.py`): shell frontend that generates 43 subcommands from `TOOLS` by introspection. All-flags (no positionals). `list[dict]` params take a JSON string. Sets `GSC_NO_BROWSER=1` at startup to prevent accidental OAuth browser popups.

**Auth layer** (`auth.py`): Three separate credential pairs (GSC API `searchconsole/v1`, Indexing API `indexing/v3`, GA4 `analytics.readonly`). Resolution order: if `GSC_SERVICE_ACCOUNT_PATH` is set, use service account credentials. Otherwise, fall through to OAuth with token cached as JSON (not pickle) at the OS user data dir (`~/Library/Application Support/gsc-mcp/` on macOS). Token files are written `chmod 0o600`; the directory is created with `0o700`. `get_ga4_property_id(override=None)` accepts an optional override string that bypasses `GA4_PROPERTY_ID`, enabling per-call multi-property support.

**Tools** (`src/gsc_mcp/tools/`): Ten modules, each owns a logical domain:
- `analytics.py`: 6 GSC search analytics tools
- `seo.py`: 6 SEO intelligence tools (quick wins, drops, cannibalization, anomalies)
- `inspection.py`: URL inspection + batch + issue categorization
- `indexing.py`: Indexing API (submit_url, submit_batch)
- `sitemaps.py`: sitemap management + `sitemap_audit` (defusedxml, SSRF-safe)
- `properties.py`: list/get GSC properties, `get_capabilities`
- `ga4.py`: 6 GA4 tools, all accept `hostname` and `country` filters via `_build_dimension_filter`
- `cross.py`: 2 cross-platform GSC+GA4 tools (also accept `hostname` and `country`)
- `crux.py`: 2 CrUX tools (Core Web Vitals via Chrome UX Report API)
- `technical.py`: `schema_validate` (JSON-LD extraction + validation, no auth needed)

**CrUX tools** (`crux.py`): `crux_page_vitals` and `crux_history` call the Chrome UX Report API via `httpx` (POST to `:queryRecord` / `:queryHistoryRecord`). Require `CRUX_API_KEY` (a plain Google API key, not a service account). The Chrome UX Report API must be enabled in the GCP project. A 404 from the API means not enough field data for that URL, returned as `verdict="not_enough_data"`.

**sitemap_audit** (`sitemaps.py`): fetches a sitemap via `httpx`, parses XML with `defusedxml.ElementTree` (prevents XXE and billion-laughs). Handles sitemap index files with one level of recursion; child sitemap URLs are validated against the parent's origin before fetching (`follow_redirects=False`, SSRF protection). Cross-references declared URLs against 90 days of GSC data. Returns `missing_sample` capped at 20 URLs. Verdicts: `empty` | `fetch_error` | `partial` (>20% URLs absent from GSC) | `healthy`.

**schema_validate** (`technical.py`): fetches any public URL, extracts `<script type="application/ld+json">` blocks with `html.parser` (stdlib), validates required fields per schema type (Article, LocalBusiness, FAQPage, Product, WebSite, BreadcrumbList, SoftwareApplication), and suggests missing schemas from URL path patterns (`/faq` → FAQPage, `/blog/` → BlogPosting, etc.). No auth required.

**Cross-platform pattern** (`cross.py`): `traffic_health_check` and `page_analysis` compose functions from `analytics.py` and `ga4.py`. They call those functions, parse their JSON strings with `json.loads`, then join on `_normalize_url` (strips scheme/host/query/trailing-slash so GSC absolute URLs and GA4 paths match). `engagement_rate` is derived as `engaged_sessions/sessions`.

**Output contract** (`meta.py`): Every tool wraps its response dict with `with_meta(data, tool=..., params=...)`, which appends a `_meta` block (`{"tool": "<name>", "params": {...}}`). Data keys are spread at the top level (not nested under `"data"`). Any new tool must follow this pattern.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FlorianBruniaux/google-search-console-mcp](https://github.com/FlorianBruniaux/google-search-console-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
