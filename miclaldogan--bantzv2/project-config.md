---
trigger: always_on
description: Persistent notes for Claude Code sessions working on this repo. Read this first.
---

# CLAUDE.md — operational reference for bantzv2

Persistent notes for Claude Code sessions working on this repo. Read this first.

## Architecture
- **bantzv2** — main repo (this directory, `/home/misa/bantzv2`). Origin: `github.com/miclaldogan/bantzv2`.
- **bantz-web** — git submodule at `vendor/bantz-web` (origin `github.com/miclaldogan/bantz-web`). A standalone search/research/news pipeline, wired into the tool layer in-process (no HTTP, no subprocess) via `src/bantz/tools/web_search.py`.
- **bantz-ui** — Tauri v2 + React desktop app at `bantz-ui/` ("Operations Center"). 6 pages: Broadcast Channel, Vitals, Kernel Log, Directives, Anomaly Watch, Settings. Talks to the daemon over WebSocket.

## Runtime / deployment
- **Daemon**: systemd **user** unit `bantz-daemon.service` (`~/.config/systemd/user/bantz-daemon.service`). Manage with `systemctl --user {restart,status,stop} bantz-daemon`. Enabled for boot; `Restart=on-failure`. Runs `python -m bantz --daemon`, foreground (Type=simple), `WorkingDirectory=~/.local/share/bantz/src`.
  - Do NOT hand-launch the daemon with `setsid`/`nohup` — it races the systemd unit and `bantz --ui`'s auto-spawn and crash-loops on the port. Use systemd.
- **Install clone**: `~/.local/share/bantz/src` — this is the editable install the venv (`~/.local/share/bantz/venv`) and daemon actually import (NOT this dev repo). It does **not** auto-pull. **After every push, sync it:**
  ```
  git -C ~/.local/share/bantz/src pull --recurse-submodules
  git -C ~/.local/share/bantz/src submodule update --init --recursive   # if vendor/ changed
  systemctl --user restart bantz-daemon                                  # for backend changes
  ```
  Submodule gotcha: a fresh pull may leave `vendor/bantz-web` uninitialized (`git submodule status` shows a leading `-`) → run the `submodule update` above.
- **WS server**: `ws://localhost:8765` (`src/bantz/interface/ws_server.py`). The UI dev server is Tauri + vite (separate ports); vite HMR picks up `bantz-ui/` changes from the install clone after a pull.
- **OAuth redirect**: port **8766** (`src/bantz/auth/google_oauth.py`, changed from 8765 to avoid the WS-server conflict). User must add `http://localhost:8766` as an authorized redirect URI in Google Cloud Console.
- **LLM providers**: ollama (local, default) | claude | gemini | openai. Selected via `BANTZ_LLM_PROVIDER`, switchable live in Settings → routed by `src/bantz/llm/router.py`. Provider clients read config **live** (model changes apply without restart); long-lived subsystems (voice, wake word) still need a restart.
- **Tokens**: `~/.local/share/bantz/tokens/` (calendar_token.json, gmail_token.json, credentials.json). Telegram bot `@bantzclaw_bot`, creds in parent `bantzv2/.env` (`TELEGRAM_BOT_TOKEN`, `TELEGRAM_ALLOWED_USERS`).
- **Location**: `core/location.py` source order: `.env` config → **live phone GPS** → wifi-SSID/places → **cached phone GPS** (7d) → **WiFi-BSSID geolocation** (nmcli→beaconDB) → GeoClue2 → ipinfo → `unknown` (display "location unknown", no wrong-city guess). **GeoIP is unreliable in Turkey** (ISPs route via Istanbul; geoclue here returns Istanbul/Kayseri/even Toronto) — so phone GPS is the only trustworthy source for the user (Elazığ). Phone GPS comes from `gps_server` (HTTP :9777 + ntfy relay, started by the daemon); user opens the page on their phone and taps send → fix is reverse-geocoded to a real city and cached in `~/.local/share/bantz/gps_city.json` so it sticks. WiFi-BSSID geolocation hits **beaconDB** (free MLS successor, no key) but returns 404 for the user's APs (unmapped). GeoClue2 (via `gi.repository` `Geoclue.Simple`) is now a low-priority fallback. `geoclue.service` is a **static, D-Bus-activated** unit: it can't be `systemctl enable`d, it auto-starts on demand (correct). **Venv gotcha**: the daemon venv (miniforge-based, `include-system-site-packages=false`) has **no `gi`** by default → geoclue silently falls through to IP. Fixed by symlinking miniforge's `gi` into the venv: `ln -sf /home/misa/miniforge3/lib/python3.13/site-packages/gi ~/.local/share/bantz/venv/lib/python3.13/site-packages/gi` (same py 3.13.13, ABI-compatible). **If the venv is rebuilt, re-create this symlink** or geoclue breaks. Verify: `~/.local/share/bantz/venv/bin/python -c "import gi"`.

## Key files changed in recent sessions
- `core/finalizer.py` — FACTS grounding block; `strip_internal()` (`<thinking>` + `[CONTEXT:...]` strip, preserves tool formatting) used by `strip_markdown` and applied to user-facing short/verbatim output in `brain.py`.
- `tools/gmail.py` — `build_query` default capped to `label:unread newer_than:7d`; `_summary` groups by `categorize()` (personal/institutional/services/payments/notifications) with emoji headers; `briefing=True` filter.
- `tools/calendar.py` — `_create_sync` logs+raises instead of swallowing to `""`; `_create` surfaces the real error as a ToolResult.
- `tools/web_search.py` — bantz-web integration: `web_search` (quick), `web_research` (deep, async, progress streamed to chat via `chat_token` bus event + cancel), `web_news`. Appends `vendor/bantz-web` to `sys.path` (append, NOT insert(0) — avoids shadowing `telegram` pkg); neutralizes bantz-web `git_commit`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [miclaldogan/bantzv2](https://github.com/miclaldogan/bantzv2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
