---
trigger: always_on
description: **HydroponicOne** is a production-grade IoT hydroponic control platform consisting of:
---

# HydroponicOne AI Coding Agent Instructions

## Project Overview

**HydroponicOne** is a production-grade IoT hydroponic control platform consisting of:
- **Firmware**: ESP32 nodes running custom C++ with deep-sleep, sensors (DHT11, DS18B20, HC-SR04, etc.), MQTT, and OTA support
- **Backend**: Node.js/TypeScript REST API + MQTT bridge + WebSocket real-time telemetry ingestion
- **Frontend**: (Not yet implemented) React/Vue dashboard
- **Docker**: Local dev environment with Mosquitto MQTT broker

## Architecture at a Glance

```
ESP32 Devices (firmware/) 
    ↓ MQTT (TLS) over broker
Backend (backend/)
    ├─ MQTT Worker: Subscribes nfthydro1337/+/# → parses → writes to InfluxDB + broadcasts via Socket.IO
    ├─ REST API: Device control, config, OTA, telemetry history
    ├─ OTA Service: Signs firmware with RSA-SHA256 + publishes to cmd/ota
    ├─ Database: PostgreSQL (devices, system config, actuation logs)
    └─ Telemetry: InfluxDB time-series measurements
Dashboard (frontend/)
    ↓ Socket.IO (real-time) + REST (historical)
```

## Critical Patterns & Conventions

### 1. MQTT Topic Hierarchy and Naming
- **Base topic** (from env): `MQTT_BASE_TOPIC` (default: `"HydroponicOne"`)
- **Sensor telemetry topics** (published by device): `{BASE_TOPIC}/{DEVICE_ID}/sensors/{sensor_type}/{sensor_name}` or `{BASE_TOPIC}/{DEVICE_ID}/power/{power_type}`
  - Example: `HydroponicOne/HydroNode_01/sensors/water/temperature`
  - Example: `HydroponicOne/HydroNode_01/power/battery`
  - Payloads: Numeric (e.g., `22.5`) or JSON `{"value": 22.5}`
- **Commands published by backend**: `{BASE_TOPIC}/{DEVICE_ID}/cmd/{command}`
  - Examples: `cmd/pump`, `cmd/mode`, `cmd/config`, `cmd/ota`, `cmd/tank`
- **Device lifecycle**: `status` and `heartbeat` topics are NOT stored as telemetry; used for device status updates

**Key implication**: All MQTT topic strings must reference `env.MQTT_BASE_TOPIC` (not hardcoded). Device IDs and sensor tags are extracted via topic parsing (see `mqtt.service.ts` lines 47–87).

### 2. Validation & Security Model
- **Environment variables**: All required env vars validated via Zod at startup in `config/env.ts` → kills process if invalid
- **Request body validation**: All API handlers use Zod schema parsing (e.g., `api.routes.ts` line 15: `z.object({...}).parse(req.body)`)
- **MQTT payload parsing**: Permissive but careful — tries JSON first, falls back to numeric parsing. Sensor values validated with `Number.isFinite()` before writing to InfluxDB (see `mqtt.service.ts` lines 65–72)
- **Never trust device input**: MQTT payloads from sensors are normalized and logged; malformed messages log warnings but don't crash the worker

### 3. Telemetry Data Flow (MQTT → InfluxDB)
1. Device publishes numeric value to `{BASE_TOPIC}/{DEVICE_ID}/sensors/{sensorType}/{sensorName}` → `mqtt.service.ts` receives (line 34)
2. Topic split into parts and device_id extracted (lines 47–55)
3. Payload parsed to number, validated for finiteness (lines 65–72)
4. Sensor tag constructed: `sensor = ${sensorCategory}_${sensorName}` (e.g., `water_temperature`) (line 87)
5. Call `writeTelemetry(deviceId, sensorCategory, sensorName, value)` → writes to **InfluxDB** measurement `hydro_telemetry` with tags `device_id`, `sensor` and float field `value` (see `telemetry.service.ts` lines 28–35)
6. Broadcast to connected WebSocket clients via `broadcastTelemetry()` (line 102, uses `sockets/socket.ts`)
7. Ignore `/status` and `/heartbeat` topics (they don't contain numeric sensor values) (lines 59–62)

**Example InfluxDB structure** (simplified schema):
```
measurement: hydro_telemetry
tags:
  device_id=HydroNode_01
  sensor=water_temperature
fields:
  value=22.4 (float)
```

**Example InfluxDB Flux query**:
```
from(bucket:"hydro_bucket")
  |> range(start: -24h)
  |> filter(fn: (r) => r.device_id == "HydroNode_01" and r.sensor == "water_temperature")
```

### 4. OTA Firmware Signing Workflow
Located in `services/ota.service.ts`.
1. Receive device_id, firmware binary path, version, download URL from REST API (`/ota/deploy`)
2. Load firmware .bin file, compute MD5 hash
3. Load private key from `OTA_PRIVATE_KEY_PATH` (env var)
4. Build payload string: `"{url}|{version}|{md5}"`
5. Sign with RSA-SHA256 using private key (base64 encoded result)
6. Publish JSON to `{BASE_TOPIC}/{device_id}/cmd/ota`:
   ```json
   { "url": "...", "version": "...", "md5": "...", "signature": "..." }
   ```
7. Device firmware verifies signature using public key before flashing

**Security note**: Protect `OTA_PRIVATE_KEY_PATH` in production. Keys stored in `firmware/data/` folder.

### 5. Database Access Pattern
- **ORM**: Prisma 7.x with PostgreSQL adapter (see `prisma/schema.prisma`)
- **Adapter initialization**: Uses `@prisma/adapter-pg` for connection pooling (see `api.routes.ts` lines 14–16):
  ```typescript
  const pool = new pg.Pool({ connectionString: env.DATABASE_URL })
  const adapter = new PrismaPg(pool)
  const prisma = new PrismaClient({ adapter } as any)
  ```
- **Config location**: Database URL moved to `prisma.config.ts` (Prisma 7 requirement; schema.prisma has no `url`)
- **Per-route client**: Single `PrismaClient` instance per route file (reused across requests)
- **Models**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [40rbidd3n/Hydro0x01](https://github.com/40rbidd3n/Hydro0x01) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
