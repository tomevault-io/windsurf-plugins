---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Waybar widgets that show usage/balance for **Claude Code, OpenAI Codex CLI, GitHub Copilot, OpenCode Zen, and Z.ai**. Each provider is one Python script that prints either a human-readable line (CLI) or a Waybar JSON blob (`--waybar`). Cron-style behavior comes from Waybar itself, not the scripts.

## Mental model

This is a **fan-in** layout, not a framework:

- N small, mostly-independent provider scripts at the repo root (`claude.py`, `codex.py`, `copilot.py`, `zen.py`, `zai.py`)
- One shared utility module (`common.py`) holding the cookie loader, file cache, output formatter, and reset-time formatter
- One fat orchestrator (`waybar_ai_usage.py`, ~900 lines) that knows about all providers and writes/restores the user's Waybar config + CSS

A provider script never imports another provider — it only depends on `common.py`. The orchestrator depends on **knowledge** of every provider (in five places — see "Adding a provider"), but not on imports of them.

## Code shape

| File | Lines | Role |
|------|-------|------|
| `common.py` | ~340 | Contract layer: `load_cookies`, `get_cached_or_fetch`, `format_eta`, `format_output`, `parse_window_*`, `open_login_url` |
| `waybar_ai_usage.py` | ~920 | Setup/cleanup/restore CLI. Reads/writes Waybar's JSON5 config and CSS, handles backups, drives the TUI module picker |
| `claude.py` / `codex.py` | ~290 each | Cookie-auth providers (claude.ai, chatgpt.com) — most code is API request + `parse_window_*` glue |
| `zen.py` | ~230 | Cookie-auth provider (opencode.ai), parses balance from rendered HTML/JS state |
| `copilot.py` | ~280 | PAT-auth via `urllib` (no `curl_cffi`); PR #13 adds Chrome-cookie HTML-scrape fallback for org-managed accounts |
| `zai.py` | ~320 | API-token-auth via `urllib`; JWT must be copied from DevTools (no browser auto-detect possible) |

`waybar_ai_usage.py` is the most complex file. It does JSON5 round-tripping, region-marker-based CSS edits, dry-run previews, and config restoration from `.bak` files. Most maintenance work touches the providers and `common.py`; `waybar_ai_usage.py` only changes when adding/removing a provider or tweaking the setup UX.

## The provider contract

Every provider script must:

1. Define a config object (cookies dict or `~/.config/waybar-ai-usage/<name>.conf`)
2. Have an `argparse` `main()` that supports `--waybar` and `--config <path>` (and `--browser` for cookie-auth ones)
3. Wrap its fetch in `get_cached_or_fetch("<name>", fetch_fn, ttl=...)` from `common.py`
4. Format output with `common.format_output(format_string, data)` so the user's `--format` flag works
5. Classify errors as **Auth Err** (HTTP 401/403/404, cookie load failures) or **Net Err** (timeouts, DNS, anything else)
6. In `--waybar` mode, **always `sys.exit(0)`** even on error — non-zero exit hides the widget entirely
7. On HTTP auth errors, call `open_login_url()` from `common.py` (no-op outside `--waybar`, has 10-min cooldown via marker file)

Reuse from `common.py` rather than rolling your own — `format_eta` accepts `str | int | None` (ISO 8601 string, Unix timestamp seconds, or null), so you should not pre-convert timestamps.

## Three auth strategies

The choice depends on what the upstream service exposes. In rough order of preference:

| Strategy | Used by | Picked when |
|---|---|---|
| **Browser cookie auto-detect** | `claude.py`, `codex.py`, `zen.py` | Cookies persist on disk and the API accepts them. Multi-browser fallback via `DEFAULT_BROWSERS` order in `common.py` |
| **Static API token in config file** | `copilot.py` (PAT), `zai.py` (JWT) | When auth is short-lived/JS-generated and not in cookies, OR when the user must explicitly grant a scoped token |
| **HTML scrape with cookies** | `copilot.py` org-managed fallback (PR #13) | Last resort when the API doesn't expose the data but the settings page does |

`curl_cffi` with `impersonate="chrome"` is required for any cookie-authenticated call to providers behind Cloudflare (claude.ai, chatgpt.com, opencode.ai, github.com settings page). Stdlib `urllib` is fine for plain API calls (Copilot billing, Z.ai).

## Cross-cutting concerns

**Caching** (`common.get_cached_or_fetch`)
- File-based: `~/.cache/waybar-ai-usage/<name>.json` with TTL (default 60s, Z.ai uses 120s)
- Multi-Waybar coordination via `.updating` marker files. Multiple monitors → multiple Waybar instances → concurrent fetches; the marker makes one win and the others wait up to 3s for the cache to land
- Fragile state to be aware of: corrupted cache files are silently treated as miss

**Error UX**
- Two buckets: `Auth Err` and `Net Err`. Auth errors trigger `xdg-open` to the provider's login page (cooldown marker at `~/.cache/waybar-ai-usage/<name>.login_opened`)
- Tooltip carries the full error message; the bar text is short (`Auth Err` / `Net Err` / icon + percentage)

**Retry**
- Cookie-auth providers (`claude.py`, `codex.py`, `zen.py`) retry once on failure (2 attempts, 10s timeout each)
- Token-auth providers (`copilot.py`, `zai.py`) do not retry. Single 10s timeout

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NihilDigit/waybar-ai-usage](https://github.com/NihilDigit/waybar-ai-usage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
