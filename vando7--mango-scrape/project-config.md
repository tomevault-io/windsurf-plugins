---
trigger: always_on
description: Personal-use "deep dive" scraper for local LLMs. Takes URLs, returns clean markdown + metadata. Intended workflow: a local model gets search snippets, then calls `deep_dive` on results it wants to read in full.
---

# AGENTS.md

Personal-use "deep dive" scraper for local LLMs. Takes URLs, returns clean markdown + metadata. Intended workflow: a local model gets search snippets, then calls `deep_dive` on results it wants to read in full.

## Architecture

```
LM Studio
  │  spawns via stdio
  ▼
mcp_shim.py (thin, self-bootstraps) ──HTTP──▶ localhost:8765
                                                  │
                                                  ▼
                                            server.py (FastAPI)
                                                  │
                                                  ▼
                                            scraper.py (patchright + trafilatura)
```

The shim auto-starts the FastAPI server on port 8765 if it isn't already running. All deps live in `.venv`. The shim is intentionally thin: stdio MCP in, HTTP POST out.

## Files

| Path | Role |
|---|---|
| `scraper.py` | Scraping logic. `_compact`, `launch_browser`, `_scroll_page`, `_extract_links`, `_extract_reddit_comments`, `scrape_one`, `scrape_many`, `screenshot_one/many`, `download_file`, `clone_repo`, `list_files`, `cat_file`, `hn_search`, `reddit_search`. YouTube: delegates transcript to `yt_transcript`. Reddit URL scraping: www→old→m.reddit fallback chain (www first for SSR post body). Post-body detection JS check before committing to a URL. Improved comment selectors for Reddit's current DOM. |
| `server.py` | FastAPI wrapper. Lifespan owns a shared browser. Endpoints: `POST /scrape`, `/screenshot`, `/scrape_youtube`, `/transcript_page`, `/download`, `/clone`, `/list`, `/cat`, `/search`, `/hn_search`, `/reddit_search`, plus `GET /health`. Brave search via `BRAVE_API_KEY`. Idle shutdown after `DEEP_DIVE_IDLE_TIMEOUT` seconds (default 300). Writes `server.pid` on startup. Listens on `8765`. |
| `yt_transcript.py` | YouTube transcript fetching with SQLite cache and word-based pagination (500-word pages). Functions: `get_transcript`, `paginate_transcript`, `get_page`, `cache_get`, `cache_put`. Cache path defaults to `./cache/yt_transcripts.db` next to the file (overridable via `DEEP_DIVE_CACHE_DB`). |
| `shim/mcp_shim.py` | stdio MCP server. `_fmt` formats a single dict as flat key=value lines; `_fmt_list` formats `{status, query, num_results, results}` shapes with `===` between blocks. Multi-line values keep real newlines — no escape artifacts. `_post()` does the HTTP call with one auto-restart-and-retry on `ConnectError` (raises `ServiceError` on persistent failure). `_ensure_server()` only auto-starts when the target host is local. |
| `shim/pyproject.toml` | Shim deps: `mcp`, `httpx`. |
| `pyproject.toml` | Host-side dev deps (for running `server.py`). |
| `README.md` | Human-facing setup. |

## Commands

```bash
# One-shot: shim auto-starts the server on first call
uv run python shim/mcp_shim.py

# Manual: start server only (shim will do it automatically anyway)
cd C:\software\searchmcp\scraper\scrape
uv sync                          # installs deps + managed Python into .venv
uv run patchright install chromium  # downloads patched Chromium binary
uv run uvicorn server:app --port 8765

# Smoke tests (server must be running)
curl -s http://localhost:8765/health
curl -s -X POST http://localhost:8765/scrape -H 'Content-Type: application/json' \
  -d '{"urls":["https://example.com"]}' | jq .

# Stop (if started manually)
taskkill /F /IM uv.exe
```

## Conventions and gotchas

**Stdio MCP hygiene (shim).** The shim speaks JSON-RPC over stdout. Never `print()` in `mcp_shim.py` or anything it imports. All logging to stderr (`logging.basicConfig(stream=sys.stderr, ...)`). A stray stdout byte corrupts the protocol and the client closes the connection silently.

**Use patchright, not playwright.** Import is `from patchright.async_api import …`. Patchright patches the `Runtime.enable` CDP leak that regular Playwright (and `playwright-stealth`) can't mask. Chromium-only. Do not "upgrade" to vanilla Playwright — you lose stealth.

**Patchright's Chromium is separate.** `pip install patchright` does not fetch a browser. `patchright install chromium` downloads a patched build distinct from any existing Playwright browser. Run once: `uv run patchright install chromium`.

**Shared browser, fresh context per URL.** `server.py` launches one browser in `lifespan` and reuses it. Each scrape creates a new `browser.new_context()` (cheap) and closes it. Do not launch per request (1–2 s overhead). Do not reuse contexts (leaks cookies/state).

**Errors are per-URL, never raise.** `scrape_one` catches everything and returns `{status: "error", error: ...}`. `scrape_many` uses `asyncio.gather(..., return_exceptions=True)`. Callers always get one result per input URL — preserve this contract.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Vando7/mango-scrape](https://github.com/Vando7/mango-scrape) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
