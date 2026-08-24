---
trigger: always_on
description: ClickerScriptingLibrary — iOS device automation via Nomix Clicker API
---


# ClickerScriptingLibrary

iOS device automation via Nomix Clicker API. Scripts simulate touch input and use AI screen recognition to navigate apps.

## Packaging

This is a pip-installable package (`nomix-clicker`), src-layout, built with setuptools.

```bash
pip install -e .          # local development (editable)
python3 -m build          # build wheel + sdist into dist/
```

Config is resolved at import time: env vars (`NOMIX_API_KEY`, `NOMIX_DEVICE_ID`, `NOMIX_API_URL`) take precedence, then a `config.json` in the current working directory, next to the entry script or one level above it, or the repo root for editable installs (or an explicit `NOMIX_CONFIG` path), then built-in defaults.

## Running examples

Example scripts live in `examples/` and require the package installed. Run directly:

```bash
python3 examples/notes.py
```

## Project structure

```
pyproject.toml           — package metadata + dependencies (replaces requirements.txt)
src/nomix_clicker/
  __init__.py            — public API exports (Clicker, Agent, parse_screen, actions, ...)
  clicker.py             — Clicker class (swipe, click, type)
  recognition.py         — Screen, Element, parse_screen()
  actions.py             — high-level helpers (open_app, chance_tap, post_comment, etc.)
  agent.py               — Agent class (autonomous AI task runner)
  api_helper.py          — low-level HTTP calls to Nomix API (don't use directly)
  environment.py         — config resolution (env vars > config.json > defaults)
  config_handler.py      — config.json loader with auto-reload every 300s
examples/
  notes.py               — safe reference implementation (open Notes, type a note)
  ai_agent.py            — autonomous AI agent task runner
tools/                   — device utilities (restart, wake-up, airplane mode, screen
                           viewing); may drop the device stream — not example material
config.json              — API_URL, API_KEY, DEVICE_ID (found via the config search path, gitignored)
```

## Imports

Import from the top-level package (or from submodules — both work):

```python
from nomix_clicker import (
    Clicker, Agent, Screen, Element, parse_screen,
    open_app, close_app, swipe_feed, swipe_back, is_ad, chance_tap,
    post_comment, random_sleep, find_and_click, DEVICE_ID,
)
```

## Coordinate system

HID absolute coordinates: **0–32767** on both axes. Device-independent, same for all screen sizes.

| Point | Coords |
|---|---|
| Top-left | (0, 0) |
| Center | (16383, 16383) |
| Bottom-right | (32767, 32767) |

- Swipe distance: typically 6000–8000 units
- `Element.center` and `Element.bbox` are already in this coordinate space
- `bbox` format from API: `[y_min, x_min, y_max, x_max]`

## API reference

### Clicker

```python
clicker = Clicker(device_id: str)
clicker.device_id: str                        # stored device ID

clicker.click(coords: tuple[int, int], duration: int = 100)
# Taps at coords (combined move+click in one call). duration = hold time in ms.

clicker.swipe(coords: tuple[int, int], up: int = 0, down: int = 0, left: int = 0, right: int = 0, duration: int = 300)
# Swipe from coords in given direction(s). Distance in HID units.

clicker.type(text: str)
# Type text on device keyboard. Max 10000 chars.

clicker.key_combo(codes: list[str])
# Press a key combination, e.g. ["MetaLeft", "Space"] for Spotlight.
# Keys pressed in order, released in reverse.

clicker.get_screenshot() -> bytes | None
# Latest JPEG frame from the device stream, or None if no frame (stream down).
# Raises requests.HTTPError on 401/403 — an auth problem, not a missing frame.
```

Under the hood `Clicker.click` calls `POST /{device_id}/tap` (combined move+click with a device-settle delay) and `swipe` calls `POST /{device_id}/move` (absolute coordinate movement). Action endpoints don't raise on failure — they return `{"success": bool, "message": str}`.

### Screen recognition

```python
screen = parse_screen(device: str | Clicker, retries: int = 3, retry_delay: float = 3.0) -> Screen | None
# Calls POST /{device_id}/screen-state. Timeout: 60s per attempt; transient
# errors are retried up to `retries` times with `retry_delay`s between attempts.
# Uses AI vision to parse the current screen into structured elements.
# Returns None on network/timeout errors (no try/except needed in scripts).
# Auth/quota errors (401/403/429) are not retried — fix the key or quota.

screen.app_name: str           # e.g. "Calculator"
screen.description: str        # natural language screen description
screen.elements: list[Element] # all detected UI elements
screen.latency: float          # API processing time in seconds

screen.find(*keywords: str, interactive_only: bool = True) -> tuple | None
# Keywords are tried in order — the first keyword matching any element wins
# (case-insensitive substring match), so earlier keywords take priority.
# Returns center coords (x, y) or None.

screen.find_and_click(clicker: Clicker, *keywords: str, interactive_only: bool = True) -> bool
# Find element and tap it. Returns True if found and tapped.

screen.contains(*keywords: str) -> bool
# Check if any keyword appears in description or any element content.
```

### Element

```python
@dataclass(frozen=True)
class Element:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nomix-ai/ClickerScriptingLibrary](https://github.com/nomix-ai/ClickerScriptingLibrary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
