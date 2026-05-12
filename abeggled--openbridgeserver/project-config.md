---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**open bridge server** is an open-source multiprotocol building automation server (MIT-licensed replacement for the proprietary Timberwolf Server). It bridges KNX, Modbus RTU/TCP, 1-Wire, MQTT, Home Assistant, ioBroker, and Zeitschaltuhr into a unified system with a FastAPI REST/WebSocket API and a Vue-based admin GUI.

## Common Commands

```bash
# Install dependencies
pip install -r requirements.txt -r requirements_dev.txt

# Run the server
python -m obs

# Run all tests
pytest tests/

# Run a single test file
pytest tests/unit/test_converter.py

# Run a specific test
pytest tests/unit/test_converter.py::test_float_to_int

# Run only adapter + unit tests (no Docker needed)
pytest tests/adapters/ tests/unit/

# Run contract tests — verify external library API surfaces (no Docker needed)
pytest tests/contracts/

# Run integration tests (requires Docker for Mosquitto)
pytest tests/integration/

# Run with coverage report
pytest tests/ --cov=obs --cov-report=term-missing

# Lint
ruff check .

# Format
ruff format .

# Docker Compose (full stack)
docker compose up -d

# Docker Compose (Mosquitto only — for local dev outside Docker)
docker compose up -d mosquitto

# Admin GUI dev server (proxies /api to localhost:8080)
cd gui && npm run dev
```

## Local Development Setup

### Prerequisites
- Python 3.13+, Docker Desktop, Node.js 20.19+ or 22+ (use nvm; `.nvmrc` pins v22)

### One-time setup

```bash
# 1. Create local venv
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt -r requirements_dev.txt

# 2. Config files
cp config.example.yaml config.yaml   # then edit (see below)
cp .env.example .env                  # set OBS_MQTT_PASSWORD

# 3. Frontend deps
cd gui && npm install
```

### config.yaml — required local overrides

The defaults assume a Docker deployment (`/data/obs.db`, `/mosquitto/passwd/passwd`).
For local dev, override these keys:

```yaml
mqtt:
  username: obs
  password: <value of OBS_MQTT_PASSWORD in .env>

database:
  path: <absolute-project-path>/data/obs.db

mosquitto:
  passwd_file: <absolute-project-path>/data/mosquitto/passwd
  reload_pid: null
  reload_command: null
  service_username: obs
  service_password: <same as mqtt.password>
```

**Important:** `mqtt.password` and `mosquitto.service_password` must match `OBS_MQTT_PASSWORD` in `.env`,
which is the password the Dockerized Mosquitto is initialized with. Mismatch → `MqttConnectError: Not authorized`.

### PyCharm run configurations

Shared configs live in `.run/` and are loaded automatically by PyCharm:

| Config | Description |
|---|---|
| **OBS Mosquitto (Docker)** | Starts MQTT broker via `docker compose up mosquitto` |
| **OBS Backend** | Runs `python -m obs` with project venv |
| **OBS GUI (Admin)** | Runs `npm run dev` in `gui/`, serves on `localhost:5173` |
| **OBS Full Dev Stack** | Compound — launches all three at once |

### Dev URLs

| Service | URL |
|---|---|
| Admin GUI | http://localhost:5173 |
| API docs (Swagger) | http://localhost:8080/docs |
| MQTT | localhost:1883 |

Default login: `admin` / `admin`

### GUI architecture

- `gui/` — Admin GUI (Vue 3 + Vite), dev server on port 5173, built to `gui_dist/` (served by FastAPI at `/`)
- `frontend/` — Visu SPA (Vue 3 + TypeScript), built to `frontend_dist/` (served by FastAPI at `/visu`)
- Both proxy `/api` to `localhost:8080` during dev via `vite.config`

## Architecture

### Startup Sequence (`obs/main.py`)

Services initialize in this fixed order (reverse-order shutdown):
1. SQLite DB + migrations
2. EventBus (async pub/sub)
3. MQTT client (connects to Mosquitto)
4. DataPoint Registry (loads from DB)
5. RingBuffer
6. History plugin
7. WebSocket Manager
8. WriteRouter (MQTT `dp/{uuid}/set` → adapters; cross-protocol propagation)
9. Protocol adapters (`@register` decorator triggers self-registration on import)
10. Logic Engine

### Core Data Flow

```
Protocol Adapter → DataValueEvent (EventBus) → [Registry, RingBuffer, History, WebSocket, WriteRouter]
                                                       ↓
                                               DEST/BOTH Bindings → other adapters
```

- **DataPoint**: a named, typed value with a stable UUID. MQTT topic `dp/{uuid}/value`.
- **AdapterBinding**: connects a DataPoint to a protocol endpoint. `direction` is `SOURCE` (reads), `DEST` (writes), or `BOTH`.
- **WriteRouter**: when a SOURCE/BOTH binding fires a `DataValueEvent`, the router automatically writes to all DEST/BOTH bindings of the same DataPoint (skipping the originating binding to prevent loops). Also handles `dp/{uuid}/set` MQTT writes from external clients.

### Registry Pattern

All major subsystems use the same singleton pattern:
- `init_*()` constructs and stores a module-level singleton
- `get_*()` retrieves it (raises `RuntimeError` if not initialized)

Adapters self-register at import time via `@register` (from `obs/adapters/registry.py`). The adapter import block in `obs/main.py` is therefore the authoritative list of enabled adapters.

### Adding a New Adapter

1. Create `obs/adapters/<name>/adapter.py`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [abeggled/openbridgeserver](https://github.com/abeggled/openbridgeserver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
