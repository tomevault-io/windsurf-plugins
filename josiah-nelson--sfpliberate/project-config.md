---
trigger: always_on
description: SFPLiberate is a Web Bluetooth companion app for the Ubiquiti SFP Wizard (UACC-SFP-Wizard) that captures SFP/SFP+ module EEPROM data over BLE, stores it in a local library, and aims to enable cloning/reprogramming modules. The architecture is a **browser-based BLE client** (vanilla JS) + **Dockerized Python FastAPI backend** with SQLite storage, served through NGINX reverse proxy.
---

# SFPLiberate – AI Coding Agent Instructions

## Project Overview
SFPLiberate is a Web Bluetooth companion app for the Ubiquiti SFP Wizard (UACC-SFP-Wizard) that captures SFP/SFP+ module EEPROM data over BLE, stores it in a local library, and aims to enable cloning/reprogramming modules. The architecture is a **browser-based BLE client** (vanilla JS) + **Dockerized Python FastAPI backend** with SQLite storage, served through NGINX reverse proxy.

## Architecture & Data Flow

### Single-Origin Design
- NGINX container serves static frontend at `/` and reverse-proxies `/api/*` to backend container
- Frontend uses relative URL `/api` (no CORS needed in production; dev CORS middleware enabled on backend)
- Backend exposes on port 80 internally; frontend NGINX exposes port 8080 externally
- Start with: `docker-compose up --build` → `http://localhost:8080`

### BLE Communication Pattern
- **Critical**: Core read/write operations occur **on-device**; BLE primarily broadcasts logs/data for capture
- Device broadcasts human-readable status logs (`sysmon: ... sfp:[x]`) and binary EEPROM dumps over BLE characteristics
- Frontend uses Web Bluetooth API (`navigator.bluetooth`) to connect and subscribe to notifications
- Write functionality is **placeholder only**—requires reverse-engineering actual BLE command protocol from official app

### Data Storage & Deduplication
- Backend stores modules in SQLite with SHA-256 checksum (`sha256` column with unique index)
- `database_manager.add_module()` returns `(id, is_duplicate)` tuple; duplicates reuse existing ID
- EEPROM stored as BLOB; metadata (vendor/model/serial) parsed server-side via `sfp_parser.parse_sfp_data()`

## Key Files & Responsibilities

### Frontend (`frontend/`)
- **`script.js`**: BLE state machine, notification handler, SFF-8472 EEPROM parser, API client
  - `handleNotifications()`: Core dispatcher—heuristic text vs binary detection for incoming BLE data
  - `parseAndDisplaySfpData()`: Client-side SFF-8472 parser (bytes 20-36 vendor, 40-56 model, 68-84 serial)
  - Safari compatibility: `acceptAllDevices` fallback, `DataView → Uint8Array` conversion for broad support
- **`index.html`**: Static UI with status indicators (`bleStatus`, `sfpStatus`), module library list, placeholder community sections
- **`nginx.conf`**: Reverse proxy config—`location /api/` passes to `http://backend:80`

### Backend (`backend/`)
- **`main.py`**: FastAPI app with 6 endpoints:
  - `GET /api/modules` → list all (excludes BLOB)
  - `POST /api/modules` → save new module (Base64 EEPROM in payload)
  - `GET /api/modules/{id}/eeprom` → raw binary BLOB (`application/octet-stream`)
  - `DELETE /api/modules/{id}` → delete module
  - `POST /api/submissions` → community inbox (writes to disk: `{uuid}/eeprom.bin` + `metadata.json`)
- **`database_manager.py`**: SQLite wrapper with SHA-256 duplicate detection, `setup_database()` migration logic
- **`sfp_parser.py`**: SFF-8472 spec parser (identical logic to frontend, validates server-side)

### Docker & Infra
- **`docker-compose.yml`**: Two services (`backend`, `frontend`); backend volume mounted at `/app/data` for SQLite persistence
- Backend env: `DATABASE_FILE=/app/data/sfp_library.db`, `SUBMISSIONS_DIR=/app/data/submissions`
- Dependencies: `fastapi`, `uvicorn[standard]`, `pydantic` (Python 3.11+)

## Development Workflows

### Local Iteration
```bash
docker-compose up --build    # Full stack at localhost:8080
docker-compose logs -f backend  # Backend logs (FastAPI + SQLite)
```
- API docs auto-generated: `http://localhost:8080/api/docs` (FastAPI Swagger)
- Frontend changes: rebuild frontend container or mount volume for live edits
- Backend changes: restart backend service or use `--reload` in Dockerfile (dev only)

### Testing BLE (requires hardware)
1. Power on SFP Wizard with module inserted
2. Open app in Chrome/Edge (Safari has limited Web Bluetooth support—see feature detection in `script.js`)
3. Click "Connect to Device" → browser shows device picker (UUID filters may fail on Safari, code falls back to `acceptAllDevices`)
4. Trigger read **on device** (button press)—app listens for binary broadcast via `notifyCharacteristic`
5. Parsed data appears in UI; click "Save" to persist to backend

### Database Schema Evolution
When adding columns to `sfp_modules` table:
- Use `database_manager.setup_database()` migration pattern (see `sha256` column addition logic with `PRAGMA table_info` check)
- **Never** break existing column contracts—add nullable columns or provide defaults

## Project-Specific Conventions

### BLE Protocol Reverse-Engineering
- **Placeholder commands** (e.g., `[POST] /sfp/write/start` in `script.js`) are **speculative guesses**—do not rely on them
- To discover actual commands: use nRF Connect app to sniff BLE traffic during official app operations
- Document findings in code comments with firmware version tested (behavior may change across updates)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [josiah-nelson/SFPLiberate](https://github.com/josiah-nelson/SFPLiberate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
