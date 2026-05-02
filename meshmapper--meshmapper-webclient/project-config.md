---
trigger: always_on
description: Handles visibility changes (release on hidden, reacquire on visible).
---

# MeshCore GOME WarDriver - AI Agent Instructions

> **Keeping This File Updated**: When you make architectural changes, add new workflows, or modify critical patterns, update this file. Ask the AI: *"Update .github/copilot-instructions.md to reflect the changes I just made"* - it will analyze the modifications and update relevant sections.

## Project Overview

Browser-based Progressive Web App for wardriving with MeshCore mesh network devices. Connects via Web Bluetooth to send GPS-tagged pings to a `#wardriving` channel, track repeater echoes, and post coverage data to the MeshMapper API for community mesh mapping.

**Tech Stack**: Vanilla JavaScript (ES6 modules), Web Bluetooth API, Geolocation API, Tailwind CSS v4

**Critical Files**:
- `content/wardrive.js` (5200+ lines) - Main application logic
- `content/device-models.json` - Device model database for auto-power selection
- `content/mc/` - MeshCore BLE protocol library (Connection classes, Packet parsing, Buffer utilities)
- `index.html` - Single-page UI with embedded Leaflet map
- `docs/` - Comprehensive workflow documentation (CONNECTION_WORKFLOW.md, PING_WORKFLOW.md, DEVICE_MODEL_MAPPING.md, etc.)

## Architecture & Data Flow

### 1. Connection Architecture
Three-layer connection system:
- **BLE Layer**: `WebBleConnection` (extends `Connection` base class) handles GATT connection, characteristic notifications
- **Protocol Layer**: `Connection.js` (2200+ lines) implements MeshCore companion protocol - packet framing, encryption, channel management, device queries
- **App Layer**: `wardrive.js` orchestrates connect/disconnect workflows with 10-step sequences (see `docs/CONNECTION_WORKFLOW.md`)

**Connect Sequence**: BLE GATT → Protocol Handshake → Device Info → Device Model Auto-Power → Time Sync → Capacity Check (API slot acquisition) → Channel Setup → GPS Init → Connected

### 2. Device Model Auto-Power Selection (NEW)
Automatic power configuration based on detected hardware:
- **Database**: `device-models.json` contains 32+ MeshCore device variants with recommended power levels
- **Detection**: `deviceQuery()` returns manufacturer string (e.g., "Ikoka Stick-E22-30dBm (Xiao_nrf52)nightly-e31c46f")
- **Parsing**: `parseDeviceModel()` strips build suffix ("nightly-COMMIT") for database matching
- **Lookup**: `findDeviceConfig()` searches database for exact/partial match
- **Auto-Set**: `autoSetPowerLevel()` configures radio power after successful deviceQuery()
- **Critical Safety**: PA amplifier models (33dBm, 30dBm) require specific input power to avoid hardware damage
- See `docs/DEVICE_MODEL_MAPPING.md` for complete architecture

### 3. Ping Lifecycle & API Queue System
Two independent data flows merge into a unified API batch queue:

**TX Flow** (Transmit):
1. User sends ping → `sendPing()` validates GPS/geofence/distance
2. Sends `@[MapperBot]<LAT LON>[ <power>]` to `#wardriving` channel via BLE
3. Starts 6-second RX listening window for repeater echoes
4. After window: posts to API queue with type "TX"
5. Triggers 3-second flush timer for real-time map updates

**RX Flow** (Receive - Passive monitoring):
1. Always-on `handleUnifiedRxLogEvent()` captures ALL incoming packets (no filtering)
2. Validates path length > 0 (must route via repeater, not direct)
3. Buffers RX events per repeater with GPS coordinates
4. Flushes to API queue on 25m movement or 30s timeout, type "RX"

**API Queue** (`apiQueue.messages[]`):
- Max 50 messages, auto-flush on size/30s timer/TX triggers
- Batch POST to `yow.meshmapper.net/wardriving-api.php`
- See `docs/FLOW_WARDRIVE_API_QUEUE_DIAGRAM.md` for visual flow

### 3. GPS & Geofencing
- **GPS Watch**: Continuous `navigator.geolocation.watchPosition()` with high accuracy
- **Freshness**: Manual pings use 60s max age, auto pings require fresh acquisition
- **Ottawa Geofence**: 150km radius from Parliament Hill (45.4215, -75.6972) - hard boundary
- **Min Distance Filter**: 25m between pings (prevents spam, separate from 25m RX batch trigger)

### 5. State Management
Global `state` object tracks:
- `connection`: Active BLE connection instance
- `wardrivingChannel`: Channel object for ping sends
- `txRxAutoRunning` / `autoTimerId` / `nextAutoPingTime`: Auto-ping state
- `lastPingLat/Lon`: For distance validation
- `cooldownEndTime`: 7-second cooldown after each ping
- `sessionId`: UUID for correlating TX/RX events per wardrive session
- `deviceModel`: Full manufacturer string from deviceQuery()
- `autoPowerSet`: Boolean tracking if power was automatically configured

**Device Model Database**: `DEVICE_MODELS` global array loaded from JSON on page load

**RX Batch Buffer**: `Map` keyed by repeater node ID → `{rxEvents: [], bufferedSince, lastFlushed, flushTimerId}`

## Critical Developer Workflows

### Build & Development
```bash
npm install                  # Install Tailwind CLI
npm run build:css            # One-time CSS build
npm run watch:css            # Watch mode for development
```

**No bundler/compiler** - Open `index.html` directly in browser (Chrome/Chromium required for Web Bluetooth).

### Debug Logging System
All debug output controlled by `DEBUG_ENABLED` flag (URL param `?debug=true` or hardcoded):
```javascript
debugLog("[TAG] message", ...args);   // General info

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MeshMapper/MeshMapper_WebClient](https://github.com/MeshMapper/MeshMapper_WebClient) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
