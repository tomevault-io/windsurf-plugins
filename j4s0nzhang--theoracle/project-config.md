---
trigger: always_on
description: Manages the pre-draft lobby and holds live `DraftArbiter` instances after sessions start.
---

# TheOracle — Project Reference

## Overview
MTG draft tracking website backend. Two core capabilities: (1) resolves OCR-scanned card identifiers to full card data via Scryfall, and (2) manages booster draft sessions end-to-end. Exposed via a FastAPI server with a Jinja2 stats website. N player apps send OCR-scanned picks to the server; the server records all state and serves live stats via browser.

## Stack
- **Language:** Python 3.12+
- **Package layout:** src layout — `src/theoracle/`
- **Build backend:** hatchling (`pyproject.toml`)
- **HTTP client:** requests
- **External API:** Scryfall (`https://api.scryfall.com`)
- **Web framework:** FastAPI + uvicorn
- **Templates:** Jinja2 (server-rendered HTML)
- **Test framework:** pytest 9+
- **Environment:** conda env named `oracle`

## Project Structure
```
src/theoracle/
    __init__.py
    card_parser.py        # OCR normalization + Scryfall lookup pipeline
    draft_arbiter.py      # MTG booster draft session manager
    db.py                 # SQLite connection factory + migration runner
    session_manager.py    # Lobby state + SessionManager registry
    main.py               # FastAPI app — API routes + website routes
    templates/
        base.html         # shared layout (Bootstrap 5)
        session_stats.html  # live draft view, auto-refreshes every 5s
        player_history.html # per-player historical picks + pack replay
        cards.html          # top 50 cards by avg pick order
migrations/
    001_initial.sql       # sessions, players, picks, results
    002_global_players.sql  # persistent player identity across sessions
tests/
    test_card_parser.py   # 47 unit tests, all mocked (no network)
    test_draft_arbiter.py # 54 unit tests, SQLite via tmp_path
    test_api.py           # 29 FastAPI tests, mocked card_parser
pyproject.toml            # dependencies, build config, pytest config
```

## Install & Run
```bash
# One-time setup (run from project root)
conda activate oracle
pip install -e ".[dev]"

# Run the server (default DB: drafts.db in cwd)
uvicorn theoracle.main:app --reload

# Override DB path
ORACLE_DB_PATH=/path/to/drafts.db uvicorn theoracle.main:app --reload

# Swagger UI (interactive API docs)
open http://localhost:8000/docs

# Run tests
pytest
```

---

## Module: `theoracle.card_parser`

### Public API
```python
from theoracle.card_parser import parse_card_identifier, CardData

card: CardData | None = parse_card_identifier("M21/123")
```

### `CardData` fields
| Field | Type | Source |
|---|---|---|
| `name` | str | Scryfall `name` |
| `mana_cost` | str | Scryfall `mana_cost` (face 0 for DFCs) |
| `type_line` | str | Scryfall `type_line` |
| `oracle_text` | str | Scryfall `oracle_text` (face 0 for DFCs) |
| `image_url` | str | `image_uris['normal']` (face 0 for DFCs) |

### Pipeline (in order)
1. **`_extract_set_and_number(raw)`** — permissive regex, accepts OCR-mangled input. Set code: 3–5 alphanumeric chars. Separator: `/`, `-`, or whitespace. Collector number: must start with a digit-like char.
2. **`_correct_collector_number(raw)`** — substitutes OCR digit-lookalikes (`O→0`, `l→1`, `I→1`, `Z→2`, `z→2`, `S→5`, `B→8`). Preserves valid Scryfall variant suffixes (`a`–`y` excluding OCR-mapped chars). Lowercase `s` is never corrected (showcase suffix).
3. **`_fuzzy_match_set(raw_set)`** — validates set code against Scryfall `/sets` list using `difflib.get_close_matches` (cutoff 0.6). Falls back to uppercased original if no match.
4. **`fetch_card(set_code, collector_number)`** — primary: `GET /cards/{set}/{number}`; fallback on 404: `GET /cards/search?q=set:X cn:Y`. Returns `None` if both miss.

### Scryfall set code cache
- Path: `src/theoracle/.scryfall_sets_cache.json`
- TTL: 30 days
- On stale/missing: fetches `GET /sets`, rewrites cache
- On network failure: degrades to stale cache data, or `[]` if no cache exists

### Error behaviour
- `RuntimeError` raised on: HTTP 429 (rate limited), HTTP 5xx (server error), network failure
- Non-404/non-200 primary responses abort without fallback (avoids misleading results)

---

## Module: `theoracle.draft_arbiter`

One `DraftArbiter` instance per live draft session. Holds all live state in RAM; persists to a SQLite database via `save()`. Thread-safe — a single `threading.RLock` serialises all mutations. Pack contents are never known up front; they are reconstructed from the ordered pick history after a pack is fully drafted.

### Architecture decisions
- **One arbiter per session** — correct granularity for a web backend (route by `session_id`, each arbiter owns its lock).
- **Session registry lives in `SessionManager`** — `session_manager.py` owns the lobby lifecycle and maps `session_id → DraftArbiter` once a session starts.
- **SQLite persistence via raw SQL migrations** — schema lives in `migrations/`; `db.py` applies migrations automatically on connection open. Migration history tracked in `schema_migrations` table.
- **Explicit round advancement** — `record_pick` never auto-advances rounds. The host calls `POST /sessions/{id}/advance` after `is_round_complete()`, giving the web layer a natural gate for "round over" UI.

### Public API
```python
from theoracle.draft_arbiter import DraftArbiter, PickEvent, PlayerStats, CardStats


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [J4s0nZhang/TheOracle](https://github.com/J4s0nZhang/TheOracle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
