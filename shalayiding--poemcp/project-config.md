---
trigger: always_on
description: MCP server that exposes Path of Exile game data to LLM agents. Two data strategies are in play:
---

# PoeMCP — Claude Code Reference

MCP server that exposes Path of Exile game data to LLM agents. Two data strategies are in play:
web scraping (poedb.tw, poewiki.net — fragile, breaks silently when the site changes) and
**parsing Path of Building Community's static game-data export** (gems, uniques, mods — much
more stable, see "Path of Building data source" below). No official API is used for game data
(GGG's Developer API is closed to new app registrations as of this writing — see "Known gotchas").

## Quick orientation

```
server.py               Entry point — FastMCP wiring, 14 tool registrations
scrapers/
  common.py             fetch_page(), Cache class, BASE_URL, HEADERS (poedb.tw scraping)
  pob_data.py            fetch_lua_file(), parse_lua_table(), parse_lua_assignments() — PathOfBuilding data
  wiki.py                fetch_wiki_page() — poewiki.net extractor
  player/                Gems (PoB), unique items (PoB), passive tree (GGG JSON), PoB build parser
  mods/                  Prefix/suffix modifier lookup by item type (PoB)
  env/                   Maps & scarabs — still poedb.tw (routed via __init__.py); maps is currently BROKEN, see gotchas
  economy/               poe.ninja pricing & currency rates + official pathofexile.com league detection
```

## Setup

```powershell
python -m venv venv
venv\Scripts\pip install -e .
```

Run server manually (for testing):
```powershell
venv\Scripts\python server.py
```

MCP config is in `.mcp.json` (gitignored — absolute paths are machine-specific).
Use `.mcp.json.example` as the template.

## Dependencies

- `mcp[cli]` — FastMCP framework
- `httpx` — HTTP client (sync and async)
- `beautifulsoup4` — HTML parsing
- Python 3.10+ required (3.13 used in venv)

Standard library: `base64`, `zlib`, `xml.etree.ElementTree`, `re`, `json`, `time`.

## Tools (14 total)

| Domain | Tools |
|--------|-------|
| Player | `search_gem`, `get_gem_detail`, `search_item`, `get_item_detail`, `search_passive`, `get_passive_detail`, `passive_tree_path`, `parse_pob` |
| Mods | `search_mods` |
| Env | `env_search`, `env_detail` |
| Economy | `price_check`, `currency_overview` |
| Universal | `fetch_wiki_page` |

Each tool function lives in its scraper module and is registered in `server.py` via `mcp.tool()`.

## Registering a new tool

1. Implement the function in the appropriate scraper module.
2. Import it in `server.py`.
3. Add `mcp.tool()(function_name)` below the relevant domain block.
4. Update `README.md` tool table.

## Data sources

| Source | URL | Used for |
|--------|-----|----------|
| **PathOfBuilding (GitHub, `dev` branch)** | `raw.githubusercontent.com/PathOfBuildingCommunity/PathOfBuilding/dev/src/Data/...` | Gems, unique items, prefix/suffix mods |
| poedb.tw | `https://poedb.tw/us` | Maps, scarabs (maps list is currently broken — page structure changed) |
| poewiki.net | `https://www.poewiki.net` | Deep mechanic explanations |
| poe.ninja | `https://poe.ninja/poe1/...` | Live pricing, currency rates |
| **pathofexile.com (official, unauthenticated)** | `www.pathofexile.com/api/leagues` | League auto-detection |
| GGG skilltree-export | GitHub raw | Passive tree node data (JSON) |

poedb.tw and the old poe.ninja/GGG-league endpoints used to cover gems/items/mods/pricing too —
they broke (site restructuring, PoE2 split) and were replaced. See git history on
`scrapers/player/items.py`, `scrapers/player/gems.py`, `scrapers/mods/item_mods.py`,
`scrapers/economy/pricing.py` for what the old scrapers looked like.

## Caching conventions

Use the `Cache` class from `scrapers/common.py`. All caches are in-memory (reset on server restart).

| Data type | TTL |
|-----------|-----|
| List pages (gems/items/maps/scarabs) | 3600 s (1 hour) |
| Mod JSON per item-type slug | 3600 s |
| Passive tree (GitHub JSON) | 3600 s |
| poe.ninja price data | 900 s (15 min) |
| League auto-detection | 3600 s |

Pattern used everywhere:
```python
_cache = Cache()
_CACHE_KEY = "all_gems"

def _get_all_gems():
    cached = _cache.get(_CACHE_KEY)
    if cached:
        return cached
    # ... fetch and parse ...
    _cache.set(_CACHE_KEY, result)
    return result
```

## Fuzzy search scoring

All `search_*` functions use the same scoring ladder:

| Score | Condition |
|-------|-----------|
| 100 | Exact name match (case-insensitive) |
| 80 | Name starts with query |
| 60 | Query substring in name |
| 50 | All query words present in name |
| 40 | Query in base type / secondary field |
| 35 | Words match across name + base type |
| 20 | Query in any string field |
| 10 | Words match across all fields |

Return top 10–20 results, sorted descending by score. Only return results with score > 0.

## Path of Building data source (gems, uniques, mods)

`scrapers/pob_data.py` fetches and parses PathOfBuilding Community's `src/Data/*.lua` files —
GGG's own game-data export, resynced by PoB maintainers every patch/league. This is why the
default branch matters:

- **Branch is `dev`, not `main`.** `git remote show origin` on the PoB repo confirms `HEAD branch: dev`.
  Using `main` 404s.
- **No pinned commit** — `POB_RAW_BASE` always points at branch HEAD. Combined with the 1-hour

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shalayiding/POEMCP](https://github.com/shalayiding/POEMCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
