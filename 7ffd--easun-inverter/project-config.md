---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Backend
```bash
cd backend
python3 -m venv .venv                  # create venv (first time only)
source .venv/bin/activate              # activate venv
pip install -r requirements.txt        # install dependencies
uvicorn main:app --reload              # dev server on :8000
```

The venv lives at `backend/.venv/` and is selected automatically by VS Code via `.vscode/settings.json`. The VS Code "Backend (uvicorn)" launch config also uses it directly.

### Frontend
```bash
cd frontend
npm run dev      # dev server on :5173 (proxies /api → :8000)
npm run build    # production build
npx tsc --noEmit # type-check only
```

### CLI (`cli/easun.py` via `cli/easun.sh`)

Purpose:
- Quick terminal tooling for inverter discovery, one-shot/live monitoring, and raw register inspection.
- Uses `backend/easunpy` directly (same protocol path as backend runtime).
- Intended for diagnostics, model reverse-engineering, and validating register maps when adding new inverter models.

Usage:
```bash
# show CLI help
./cli/easun --help

# discover inverter on LAN (prints ready-to-run templates with discovered IPs)
./cli/easun discover

# monitor (model is required); discovery-first if IP pair not provided
./cli/easun monitor --model ISOLAR_SMG_II_6K --once
./cli/easun monitor --model ISOLAR_SMG_II_6K --interval 10

# explicit IP pair (recommended for repeated runs)
./cli/easun --inverter-ip 192.168.1.160 --local-ip 192.168.1.179 monitor --model ISOLAR_SMG_II_6K --once

# raw register reads
./cli/easun read-registers 200 100
./cli/easun --inverter-ip 192.168.1.160 --local-ip 192.168.1.179 read-registers 200 100
./cli/easun read-registers 270 20 --fmt UnsignedInt --raw
```

Connection behavior:
- If both `--inverter-ip` and `--local-ip` are supplied, CLI uses them directly.
- Otherwise, CLI runs discovery for that command invocation.
- CLI does not use backend persisted `connection_config.json`.

### Docker (deploy to Linux server)
```bash
# Build for linux/amd64 (required when building on Apple Silicon)
docker build --platform linux/amd64 -t easun-inverter:latest .

# Export
docker save easun-inverter:latest | gzip > easun-inverter.tar.gz

# Copy to server and load
scp easun-inverter.tar.gz user@server:~/easun/
ssh user@server "sudo docker load < ~/easun/easun-inverter.tar.gz && sudo docker compose up -d"
```

Server-side `docker-compose.yml` (no `build:` key, just reference the loaded image):
```yaml
services:
  easun:
    image: easun-inverter:latest
    network_mode: host          # required for UDP broadcast + inverter reverse TCP
    volumes:
      - ./config:/data
    restart: unless-stopped
```

## Architecture

### Overview
FastAPI backend + React/Vite frontend. The backend imports `easunpy` from `backend/easunpy/` (copied into the project).

### Communication flow
1. **Discovery** — `ws://localhost:8000/ws/discover` WebSocket streams UDP probe attempts live (each probe is a `{"type": "trying"|"timeout"|"found"|"retry"|"stopped", ...}` message). Retries all 4 probe messages in a loop until found or client disconnects.
2. **Live data** — `ws://localhost:8000/ws/live` (frontend connects directly to `:8000`, bypassing Vite proxy). Client sends JSON config first: `{inverter_ip, local_ip, model}`. Backend polls `AsyncISolar.get_all_data()` every 20 s and streams results. On every successful poll it also calls `mqtt_manager.publish_data()` if MQTT is connected.
3. **Connection config** is persisted to `backend/connection_config.json` on first successful WS connect. On app load, `App.tsx` fetches `/api/connection-config` and skips setup if config exists.

### easunpy protocol (`backend/easunpy/`)
- `AsyncModbusClient` starts a TCP server on `local_ip:8899`; the inverter initiates a **reverse TCP connection** back to it.
- First connection: UDP broadcast to `inverter_ip:58899` tells the inverter where to connect (`set>server=<local_ip>:<port>;`). After the first successful TCP connect, `_ever_connected = True` and subsequent reconnects skip UDP — the inverter remembers the server address.
- `send_bulk` sends Modbus RTU-over-TCP commands; `AsyncISolar.get_all_data()` raises `ConnectionError` if all register reads return None.

### Persistence files (backend/config/)
Both files live under `backend/config/` locally (gitignored) and `/data/` in Docker (mounted volume). Path is controlled by the `CONFIG_DIR` env var.
- `connection_config.json` — `{inverter_ip, local_ip, model}` saved on WS connect, read on startup to auto-connect.
- `mqtt_config.json` — MQTT broker credentials, saved on successful connect, auto-reconnected on startup via FastAPI lifespan.

### MQTT / Home Assistant
`mqtt_manager.py` holds a singleton `MQTTManager`. Sensors are defined in `SENSOR_DEFS` as tuples: `(id, name, unit, device_class, state_class, icon, data_path, entity_category)`. Primary sensors have `entity_category=None`; detail sensors use `"diagnostic"`. `publish_discovery()` emits HA MQTT discovery messages; `publish_data()` publishes current values using dot-notation paths into the serialized data dict.

### Frontend pages

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [7FFD/easun_inverter](https://github.com/7FFD/easun_inverter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
