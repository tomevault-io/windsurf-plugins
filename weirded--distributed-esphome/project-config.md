---
trigger: always_on
description: Guidance for Claude Code (claude.ai/code) when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) when working in this repository.

## Project Overview

ESPHome Fleet (internally: `distributed-esphome`) manages fleets of ESPHome devices — offloads compilation to remote workers, schedules upgrades, pins versions per device, and organizes devices via tags. Runs as a Home Assistant add-on with a built-in local worker. Additional build workers run in Docker on remote machines, poll the server for jobs, compile firmware using ESPHome, and push firmware via OTA directly to ESP devices.

**Naming convention:** user-facing docs/UI/log lines say **"ESPHome Fleet"**. Code identifiers, the GitHub repo (`weirded/distributed-esphome`), Docker image names (`esphome-dist-server`, `esphome-dist-client`), the add-on slug (`esphome_dist_server`), Python module names, and the YAML comment marker (`# distributed-esphome:`) all keep their original `distributed_esphome` / `esphome-dist-*` form — changing those would force a migration on every existing install for no user benefit.

## Architecture

### Server (`ha-addon/server/`)

`aiohttp` async application with two authentication tiers:

- `/api/v1/*` — Bearer token auth for build workers (also accepts requests from the HA Supervisor IP).
- `/ui/api/*` — HA Ingress trust (no worker auth) for the browser UI.

Component responsibilities:

- `main.py` — app setup, auth middleware, background loops (timeout checker, HA entity poller, PyPI version refresher), HA Ingress `X-Ingress-Path` injection.
- `job_queue.py` — in-memory job queue persisted to `/data/queue.json`. State machine: `PENDING → WORKING → SUCCESS | FAILED | TIMED_OUT`. Jobs retry up to 3 times before permanent failure. On server restart, `WORKING` jobs reset to `PENDING`. Loader recovers gracefully from malformed/truncated queue files.
- `scanner.py` — discovers `.yaml` targets in `/config/esphome/`. `create_bundle()` produces a tar.gz of the full config directory (including `secrets.yaml`, needed for ESPHome's `!secret` resolution). **ESPHome is NOT bundled in the server Docker image (SE.1–SE.10).** At first boot, `ensure_esphome_installed()` lazy-installs the version reported by the HA ESPHome add-on into `/data/esphome-versions/<ver>/` via the shared `VersionManager`. The venv's `site-packages` is prepended to `sys.path` so `from esphome.* import ...` works; the binary at `<venv>/bin/esphome` is used by `/ui/api/validate`. Downstream callers (`_resolve_esphome_config`, `/ui/api/components`, validate) degrade gracefully while the install is in flight — 1–3 min on first boot; subsequent restarts are instant.
- `registry.py` — in-memory build worker registry (no persistence); workers are "online" if last heartbeat was within `worker_offline_threshold` seconds.
- `device_poller.py` — discovers ESPHome devices via `_esphomelib._tcp` mDNS, polls them via `aioesphomeapi` for running version.
- `api.py` — worker REST API (register, heartbeat, claim job, submit result, stream log). Parses every request body through the typed pydantic models in `protocol.py`.
- `ui_api.py` — browser JSON API (targets, devices, workers, queue, compile, cancel).
- `protocol.py` — **single source of truth** for server↔worker wire messages (pydantic v2). Byte-identical copy lives in `ha-addon/client/protocol.py`; a test enforces they match.
- `static/` — Vite-built React app output (source in `ha-addon/ui/`).

### Worker (`ha-addon/client/`)

`client.py` is a synchronous polling loop with a background heartbeat thread. Registers with the server, polls for jobs, ensures the correct ESPHome version is installed (`version_manager.py` — LRU cache of virtualenvs under `/esphome-versions/<version>/`), extracts the config bundle, runs `esphome run`, and submits results. Because the worker performs the OTA upload itself, **it must have network access to the ESP devices**.

`IMAGE_VERSION` (baked into the Docker image) and `MIN_IMAGE_VERSION` (in `ha-addon/server/constants.py`) gate the in-place source-code auto-update: the server refuses to push `.py` updates to workers whose Docker image is below `MIN_IMAGE_VERSION`, because a stale image can't be fixed by rewriting files in place.

### Job Bundle Flow

When a worker claims a job, the server calls `scanner.create_bundle()` which tarballs the ESPHome config directory into a base64 payload. The worker extracts this, compiles the target YAML, and OTA-flashes the firmware directly to the ESP device.

### Configuration

Server config is loaded from `/data/options.json` with environment variable fallbacks. Worker config is all via environment.

Key worker env vars:

| Variable | Default | Description |
|----------|---------|-------------|
| `SERVER_URL` | required | e.g. `http://homeassistant.local:8765` |
| `SERVER_TOKEN` | required | Shared auth token |
| `POLL_INTERVAL` | `5` | Seconds between job polls when idle |
| `HEARTBEAT_INTERVAL` | `10` | Seconds between heartbeats |
| `JOB_TIMEOUT` | `600` | Compile timeout in seconds |
| `OTA_TIMEOUT` | `120` | OTA upload timeout in seconds |
| `MAX_ESPHOME_VERSIONS` | `3` | Max cached ESPHome versions on disk |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [weirded/distributed-esphome](https://github.com/weirded/distributed-esphome) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
