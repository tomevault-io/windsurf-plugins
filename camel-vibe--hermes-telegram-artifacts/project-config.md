---
trigger: always_on
description: Build interactive HTML artifacts for the Telegram Mini App. Use for ANYTHING that benefits from being visual/interactive: recipes (with portion calculators), shopping lists (with localStorage), calculators, charts, diagrams, reference sheets, reports, and tools. When a user asks for food, recipes, meal planning, shopping, groceries, or anything that could be an interactive widget — build an artifact.
---


# Artifact Builder

Generate interactive HTML artifacts that render in the Telegram Mini App.

## TL;DR — 2 Steps

```bash
# 1. Write HTML to /tmp/thing.html (use template below)
# 2. Send it:
python3 ~/.hermes/skills/creative/artifact-builder/scripts/send-artifact.py \
  /tmp/thing.html "Title" <host> [chat_id] [thread_id]
```

**Host is required** (or set `HERMES_DASHBOARD_HOST` in `~/.hermes/.env`). Chat_id and thread_id can come from env vars — pass explicitly for reliability.

### Env vars in `~/.hermes/.env`

```bash
TELEGRAM_BOT_TOKEN=***             # auto-loaded (already there if Hermes is configured)
HERMES_DASHBOARD_HOST=your-host.com  # Tailscale Funnel / nginx host
HERMES_ARTIFACT_CHAT=123456789     # default chat_id (optional)
HERMES_ARTIFACT_THREAD=999         # default thread_id (optional)
```

### Resolving chat_id and thread_id

Each conversation is a topic inside one DM chat. The session context shows `thread: <N>` which is the **thread_id** (topic number), NOT the chat_id.

**Resolution order for send-artifact.py:**
1. CLI args (highest priority)
2. `HERMES_SESSION_THREAD_ID` / `HERMES_SESSION_CHAT_ID` from env (ContextVar bridge — can be stale!)
3. `HERMES_ARTIFACT_THREAD` / `HERMES_ARTIFACT_CHAT` env vars (reliable fallback)

**The thread_id from the ContextVar bridge may be wrong** when multiple DM topics are active concurrently. If the artifact lands in the wrong topic, pass thread_id explicitly from the session context header (`thread: <N>`).

**Never use the thread_id number as the chat_id.** They are different values.

## When to Use

**Use proactively.** Don't wait for the user to ask for a visualization. If a response contains 3+ numeric data points, a chart is better than prose. If explaining something interactive helps, build it.

**Always build artifacts for:**
- Any recipe → artifact with portion calculator
- Shopping lists, groceries → persistent checklist with localStorage
- Charts, data visualization → inline SVG or Chart.js
- Calculators, converters, reference tools
- Reports, comparisons, timelines, itineraries
- Architecture diagrams, flowcharts (SVG/Canvas)
- Anything where interaction > reading

**Don't build for:** Simple answers (1-2 numbers), text-only responses, quick lookups.

## Architecture

```
Agent writes HTML → artifact-server.py (port 9877) → stored on disk
                                            ↓
send-artifact.py → Bot API web_app button → https://your-host/artifact/<id>
                                            ↓
                                User taps → Mini App panel slides up
```

**Prerequisites:**
1. `HERMES_DASHBOARD_HOST` and `HERMES_ARTIFACT_CHAT` set in `~/.hermes/.env`
2. `TELEGRAM_BOT_TOKEN` auto-loaded from `~/.hermes/.env` (already there if Hermes is configured)
3. HTTPS endpoint (Tailscale Funnel, nginx, caddy, etc.)

**The agent must assume the artifact server is running.** `send-artifact.py` auto-starts it if needed — never waste a turn checking `curl localhost:9877` or `pgrep artifact-server`. Just build the HTML and send it.

**Standalone (no Hermes):** https://github.com/camel-vibe/hermes-telegram-artifacts

## How to Deliver an Artifact

### Method 1: send-artifact.py (standalone, proven)

```bash
# Pass host explicitly, or rely on HERMES_DASHBOARD_HOST env var:
python3 ~/.hermes/skills/creative/artifact-builder/scripts/send-artifact.py \
  /tmp/thing.html "Title" <host> [chat_id] [thread_id]
```

- Registers the artifact with the artifact server (port 9877)
- Sends a `web_app` button via Bot API to the target chat/thread
- **Do NOT rely on env var defaults for thread_id** — extract from session context header

### Method 2: send_message with ARTIFACT: prefix (gateway-integrated)

```python
send_message(message="Here's the report:\n\nARTIFACT:<id> Report Title", target="telegram:<chat_id>:<thread_id>")
```

- The gateway adapter strips the `ARTIFACT:<id>` tag and sends a `web_app` button on the first chunk
- Requires the gateway to be running (picks up the button on next restart)
- **The tag must NOT be the only content** — if stripped text is empty, send_message rejects it
- Less reliable than `send-artifact.py` during Tailscale Funnel flapping (gateway adds latency)

### API Endpoints

- `POST /artifact` — register: `{"title": "...", "html": "..."}` → `{"id": "...", ...}`
- `GET /artifact/<id>` — serve HTML (with TG lifecycle injections)
- `GET /artifact/latest` — serve the most recent artifact
- `GET /artifacts` — JSON list: `{"artifacts": [{id, title, type, timestamp, age}, ...]}`
- `GET /artifacts/all` — **gallery page** (HTML): latest expanded with iframe, rest collapsed, Open/Delete buttons
- `GET /artifacts/latest-age` — age in seconds of latest artifact
- `DELETE /artifact/<id>` — remove

**Note:** All endpoints are served by the standalone `artifact-server.py` (bundled with this skill). No webui patches needed.

## Starter Template

```html
<!DOCTYPE html>
<html>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [camel-vibe/hermes-telegram-artifacts](https://github.com/camel-vibe/hermes-telegram-artifacts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
