---
trigger: always_on
description: Zephyr - A Python REST API client for PTC Windchill PLM. Query parts, documents, change requests, suppliers, BOMs, CAD documents, workflows, and more. Supports OAuth 2.0 and Basic authentication.
---


# Zephyr - Windchill PLM REST API Client

Zephyr is a Python client for interacting with PTC Windchill PLM REST APIs. Supports OAuth 2.0 and Basic authentication.

---

## Capabilities Summary (Read This First)

Zephyr provides comprehensive OData query support for Windchill PLM:

**OData Filter Expressions - FULLY SUPPORTED:**
- Data types: String, Int16/32/64, Boolean, DateTimeOffset, Single, Double
- Comparison: `eq`, `ne`, `gt`, `lt`, `ge`, `le`
- Logical: `and`, `or`, `not`
- String functions: `startswith`, `endswith`, `contains`
- Type checking: `isof`
- Special properties: `ID`, `CreatedBy`, `ModifiedBy`, `View`

**Domain Clients Available:** 28 domains (ProdMgmt, DocMgmt, ChangeMgmt, QMS, CAPA, etc.)

**Response Caching - BUILT-IN:**
- Dual-backend: in-memory LRU + file-based persistence
- TTL-based expiration (default 300s, configurable per entity set)
- Auto-invalidation on write operations (POST/PATCH/DELETE)
- Cache stats tracking (hits, misses, hit rate)
- Zero external dependencies (stdlib only)

**CLI - TERMINAL INTERFACE:**
- 30+ subcommands: parts, documents, bom, search, count, domains, cache, etc.
- Auto-corrects common OData filter mistakes (enum /Value suffix)
- Tabular and JSON output formats
- Run: `python scripts/zephyr_cli.py <command> [options]`

**Async Query Manager - PARALLEL QUERIES:**
- Parallel HTTP via aiohttp with semaphore concurrency control
- Retry with exponential backoff for transient failures
- Parallel BOM traversal and batch entity fetching
- Falls back to synchronous when aiohttp is unavailable

**Key Files:**
- `AGENT_NOTES.md` - Critical learnings for AI agents (READ THIS)
- `scripts/zephyr_cli.py` - Terminal CLI for quick queries
- `scripts/async_query_manager.py` - Parallel query engine
- `scripts/odata_filter_builder.py` - OData filter construction
- `scripts/cache_manager.py` - Response caching (memory + file, TTL, LRU)
- `scripts/domains/<Domain>/client.py` - Domain-specific helpers
- `references/CAPABILITIES.md` - Full capability reference

---

## Quick Start

1. Copy `config.example.json` to `config.json` in the skill directory
2. Configure your Windchill server URL and authentication
3. Use domain clients for best practices

```python
import sys
sys.path.insert(0, '/home/ubuntu/.hermes/skills/zephyr/scripts')
from domains.ProdMgmt import ProdMgmtClient

client = ProdMgmtClient(config_path="config.json")

# Query parts (use query_entities for Parts entity set)
parts = client.query_entities('Parts', top=10)

# Search parts by term
results = client.search_parts('bracket', top=10)

# Get part by number
part = client.get_part_by_number("PART-001")
```

---

## CRITICAL WARNINGS

| Issue | Solution |
|-------|----------|
| **NEW AGENTS: Read capabilities first** | See `references/CAPABILITIES.md` for full feature reference |
| **Use ODataFilter, not raw strings** | `from odata_filter_builder import ODataFilter` |
| **scripts/old/ is DEPRECATED** | Use domain clients in `scripts/domains/` instead |
| **DO NOT create ad-hoc scripts** | Use existing domain clients directly - they support all query patterns |
| **URL double-slash bug** | Client auto-fixes trailing slashes. If "Invalid domain request", check URLs. |
| **CSRF token required** | Header: `CSRF_NONCE: <token>` (NOT `X-PTC-CSRF-Token`) |
| **OData properties case-sensitive** | Properties are PascalCase: `Number` not `number`, `Name` not `name`, `ContainerID` not `containerID`. Wrong case = 400 error. |
| **Enum properties need /Value** | PTC.EnumType props (State, Status, Priority, Severity) require `/Value`: `State/Value eq 'RELEASED'` NOT `State eq 'RELEASED'`. CLI auto-corrects. |
| **Navigation URL order** | `/{nav}` segment must come BEFORE `?$expand=` params. Wrong order = 400 error. |
| **BOM child details via Uses** | PartUse links lack child Number/Name by default. `get_bom()` auto-expands `$expand=Uses`. Read from `item['Uses']['Number']`. |
| **GetBOM not exposed** | Use `Uses` navigation: `client.get_bom(part_id)` (auto-expands child part details) |
| **Slow API (7-8s/call)** | PTC demo servers are slow - this is expected, not a bug |
| **ManufacturerParts in ProdMgmt** | ManufacturerParts/VendorParts are in ProdMgmt domain, NOT SupplierMgmt |
| **search_* uses $search (full-text)** | `search_parts()`, `search_documents()` use full-text search across ALL fields. For field-specific filtering, use `query_entities()` with `filter_expr=\"contains(Name, 'term')\"` |
| **Cache enabled by default** | Set `cache.enabled: false` in config to disable. Use `cache_clear()` to reset. |

---

## Response Caching

Zephyr includes built-in response caching that dramatically reduces latency for repeated queries. On PTC demo servers where each call takes 7-8 seconds, cached responses return in <1ms.

### How It Works

- **GET requests** are automatically cached (memory + optional file)
- **Write operations** (POST/PATCH/DELETE) auto-invalidate affected entity set
- **TTL expiration** ensures stale data is eventually refreshed
- **File persistence** survives across sessions

### Configuration

Add to `config.json`:

```json
{
  "cache": {
    "enabled": true,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [srinivasmd/windchill-plm](https://github.com/srinivasmd/windchill-plm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
