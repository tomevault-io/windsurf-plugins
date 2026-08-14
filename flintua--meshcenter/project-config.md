---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

MeshCenter is a Flask web control center for a Meshtastic LoRa radio attached to a Raspberry Pi over USB serial. It talks to the radio exclusively through the official `meshtastic` CLI (never the Meshtastic Python API directly): one long-lived `meshtastic --listen` subprocess whose stdout is parsed line-by-line with string/regex matching, plus short-lived `meshtastic --info` / `--sendtext` invocations for one-off commands. There is no database — all persistence is local JSON files (one SQLite file for waypoints) under `data/`.

## Running it

There is no build step, package.json, linter config, or test suite in this repo — don't invent `npm run`, `pytest`, or lint commands that don't exist here.

```bash
python3 -m venv --system-site-packages venv   # --system-site-packages required for Picamera2 on the Pi
source venv/bin/activate
pip install -r requirements.txt

cp config.example.py config.py                # edit MESHTASTIC_PORT / LOCAL_NODE_ID / LOCAL_NODE_NAME
mkdir -p data

python server.py                               # dev run, http://<host>:5000
# or, in production:
sudo systemctl restart meshcenter.service       # see deploy/meshcenter.service
```

`config.py` and `weather_secrets.py` are gitignored local files; `server.py` exits at import time if `config.py` is missing or missing required variables (see the `required_vars` check near the top of `server.py`). When changing code that reads config, check `config.example.py` for the authoritative variable list.

Manual verification is normally done against a real or simulated radio (`meshtastic --port <dev> --info`) and by exercising the REST endpoints/UI in a browser — there's no automated test harness to run instead.

## Architecture

### `server.py` is the core, not a thin entrypoint

`server.py` (~5,200 lines) owns the Flask `app`, nearly all shared mutable state (`nodes`, `messages`, `chats`, `settings`, locks, background threads), and most route handlers that haven't yet been split out. Newer feature areas live in `api/*.py`, but they are **not Flask Blueprints** — each is a plain function `register_<area>_routes(app, state_lock, ..., <30+ shared globals/functions>)` called from `server.py`, closing over the objects/functions passed in. When adding a new route module, follow this dependency-injection-by-parameter-list pattern rather than introducing Blueprints, and wire the new `register_*_routes(...)` call into `server.py`.

The project's own stated direction (see README "Modular Architecture") is to keep shrinking `server.py` by moving logic into `api/`, `meshsrv/`, `storage/`, `telemetry/`, `camera/` — prefer extending those modules over growing `server.py` further when the code is genuinely a separate concern.

### The radio link: subprocess + text parsing, not an SDK

`listen_meshtastic()` in `server.py` runs `meshtastic --listen` as a subprocess and classifies each stdout line by substring checks (`"NODEINFO_APP"`, `"TELEMETRY_APP"`, `"WAYPOINT_APP"`, `"TEXT_MESSAGE_APP"`, etc.), then hands multi-line blocks to parsers like `process_nodeinfo`, `parse_telemetry_from_listen_line`, `parse_waypoint_from_listen_line`. This is fragile by nature (it depends on the CLI's human-readable log format) — when the CLI output format changes across `meshtastic` package versions, these parsers are what breaks. Sending uses separate short-lived CLI calls (`meshsrv/meshsrv.py: send_text`, and the send worker in `api/api_chat.py`) that must not run concurrently with the listener on the same serial port — that's what `radio_lock`, `pause_listen` (a `threading.Event`), and `RadioConnectionManager` (`meshsrv/radio_manager.py`) coordinate. `stop_listener()` / `wait_serial_release()` / `prepare_radio_command()` in `server.py` implement the "stop listener, wait for the OS to actually free the serial device, run the one-off command, resume listener" dance — reuse them rather than calling the CLI directly from new code.

Background threads (`listen_meshtastic`, `telemetry_worker`, `radio_health_worker`, `cpu_history_worker`, the chat send-queue worker in `api/api_chat.py`) all run for the lifetime of the process. `pause_listen.is_set()` must be respected by anything that wants exclusive serial access, and `state_lock` guards the in-memory JSON-backed state (`nodes`, `messages`, `chats`) during concurrent reads/writes.

### Multi-radio profiles

MeshCenter supports switching between physical Meshtastic radios and keeps each one's data isolated:

- `meshsrv/instance_manager.py` (`instance.json`) tracks this MeshCenter installation's identity and which profile is currently active.
- `meshsrv/radio_identity.py` detects the connected radio and compares it against the configured/accepted one (`MATCH` / `MISMATCH` / `NOT_FOUND`) — the listener refuses to start (`RADIO_IDENTITY_RESULT.status != "MATCH"`) until identity is verified, to avoid silently mixing one radio's data with another's.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FlintUA/MeshCenter](https://github.com/FlintUA/MeshCenter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
