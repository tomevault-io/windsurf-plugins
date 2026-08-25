---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CS-Scout 2.0: CS2 demo analysis and **multi-round replay** visualization. The server
only parses demos and emits per-player JSON; the browser renders an animated canvas
replay. The generated map set currently contains 8 maps (Ancient, Anubis, Dust2,
Inferno, Mirage, Nuke, Overpass, and Train). Two subsystems:

1. **Web server** (`server/`) — Flask VPS server that accepts 5E usernames + a map,
   auto-fetches demos, parses them into per-player replay JSON, and serves a canvas UI.
2. **Local tools** (`tools/`, `D:/CSAI/` root) — legacy offline scripts (heatmap viewer,
   zone editor, calibrator) retained from 1.0; not part of the 2.0 server path.

---

## Web Server System (primary active work)

### Architecture

```
Browser UI (index.html + static/app.js + static/replay.js)
    │  POST /api/analyze  {usernames[], map, max_demos, key, mode?}
    ▼
web_server.py  (Flask, port 5000)
    │  mode defaults to "normal"
    ├── normal → pipeline.run(...)       (original stable producer/consumer pipeline)
    └── fast   → pipeline.run_fast(...)
          ├── concurrent discovery + ThreadPool demo downloads
          └── ProcessPool demo parsing → deterministic player JSON assembly
    ▼
output/player_{domain}.json  +  output/analysis_summary.json
    ▼
Browser fetches /api/player/<domain> → ReplayPlayer canvas (looping, unified CT/T; merged-pistol + per-player Buy)
```

### Module Map (`server/`)

| Module | Role |
|--------|------|
| `maps.py` | Runtime map loader: `load_map(name)`, `game_to_pixel(transform,gx,gy)`, `available_maps()` |
| `setup_maps.py` | One-time: pull radar.png + transform from awpy into `data/maps/<map>/` |
| `parse.py` | `get_round_table`, `classify_rounds` (Pistol/Buy/dropped on CT/T), `parse_positions`, `parse_grenades_for_rounds`, `parse_demo` (merger) |
| `combat.py` | `parse_combat_stats` (K/D + AWP-hold rate), `aggregate_combat_stats` |
| `player_json.py` | `build(...)` — assembles per-player JSON from rounds + combat |
| `pipeline.py` | stable `run(...)`, parallel `run_fast(...)`, demo dedup/single-flight, cleanup, atomic JSON output, deterministic round assembly |
| `api_client.py` | 5E scraping: `search_player`, `get_demos_by_domain(domain, map_name, count)`, `get_steamid_for_player`, `download_demo` |
| `web_server.py` | Flask endpoints + background runner |

### 5E Platform API (api_client.py)

Two base URLs: `https://arena.5eplay.com` (player search, match list) and
`https://gate.5eplay.com` (match detail / steamid extraction).

- `search_player(username)` → `(domain, matched_username)` — domain is a URL-safe ID like `0705cupvvglq`.
- `get_demos_by_domain(domain, map_name, count=10)` → `[{match_code, demo_url}]`
  - Uses one recent Arena match to resolve the player's 36-character UUID from Gate match detail.
  - Pages Gate history with `page`, `limit=30`, and the UUID (bounded to 30 pages).
  - Filters by map, deduplicates match IDs, and resolves authoritative `main.demo_url` values
    from match detail even when the list row has no URL.
  - UUID/Gate failure falls back to bounded Arena recent-match variants. A complete source
    outage raises `DemoLookupError`; a valid empty response remains an ordinary empty result.
- A shared `requests.Session` retries transient connection/read errors and HTTP 5xx responses.
- `get_steamid_for_player(match_code, username, domain)` — prefers the stable player domain,
  then falls back to username matching for older roster payloads.

### Pipeline Modes (`pipeline.py`)

**Normal mode** calls `run(...)` and preserves the original stable flow. The download thread
enqueues each `.dem` immediately after extraction
so the main thread parses it while the next downloads. Queue `maxsize=10`. Item types:
- `{"type":"demo", "i","username","domain","steamid","demos_found","dem_file","dem_idx"}`
- `{"type":"player_done", "i","username","domain","steamid","demos_found"}`
- `{"type":"player_failed", "i","username","reason"}`
- `None` — sentinel

**Fast mode** calls `run_fast(...)`:

- Player discovery and Demo downloads use independently bounded thread pools. Each completed
  download is submitted immediately to a spawn-based process pool, so CPU parsing overlaps
  remaining network work.
- Results are restored to input-player and source-Demo order before round offsets are assigned;
  worker completion timing therefore cannot change final JSON round IDs.
- A per-match single-flight lock plus a persistent-index recheck ensures players sharing a match
  perform one real download and reuse the same extracted Demo.
- Unexpected parse/build failures are isolated to their Demo or player. Player and summary JSON
  use atomic replacement so API readers never see a partially written document.

Fast-mode concurrency is configured by `CS_SCOUT_FAST_DOWNLOAD_WORKERS` and
`CS_SCOUT_FAST_PARSE_WORKERS`. Default parse fan-out is also capped against available RAM using
`CS_SCOUT_FAST_PARSE_MEMORY_PER_WORKER_MB` (2048) and
`CS_SCOUT_FAST_PARSE_MEMORY_RESERVE_MB` (1024).

Per-player progress steps (via `progress_cb(i, total, name, step, msg)`):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EytleBB/CS-Scout](https://github.com/EytleBB/CS-Scout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
