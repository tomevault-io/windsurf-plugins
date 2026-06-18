---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Plugin Overview

**Heatmiser Neo** - Indigo plugin for Heatmiser Neo thermostats and smart plugs

- **Version**: 2023.1.0 (API 3.0 / Python 3)
- **Bundle ID**: `com.racarter.indigoplugin.heatmiser-neo`
- **Author**: Alan Carter
- **Documentation**: https://carter53.wordpress.com/indigo/heatmiser/

Integrates Heatmiser Neo heating system with Indigo home automation via the Neohub central controller.

## Versioning & Release

### Version bump is required for every PR

The `PluginVersion` in `HeatmiserNeo.IndigoPlugin/Contents/Info.plist` must be bumped in every PR. CI runs a version-check that fails if the version already exists as a git tag. **Do not merge with failing checks.**

Version format: `YYYY.R.patch` (e.g. `2026.0.2`). Bump the patch for fixes/docs, minor for features.

On merge to main, the `create-release` workflow automatically creates a GitHub release with a `.zip` bundle of the plugin.

### PR checklist

1. Bump `PluginVersion` in `Info.plist`
2. Push and create PR
3. Wait for version-check CI to pass
4. Merge only after all checks are green

## Architecture

### Communication Protocol

The plugin supports two connection modes to the **Neohub** device:

**WSS — Secure WebSocket (default, recommended):**
- **Port**: 4243 (TLS, self-signed cert)
- **Protocol**: Two-layer JSON envelope over WebSocket. Outer message has `message_type` + `message` fields; inner payload contains API token and `COMMANDS` array with `COMMAND`/`COMMANDID` pairs.
- **Authentication**: API token (generated in Heatmiser Neo App → Settings → Api Access → Tokens)
- **Connection**: Fresh WebSocket opened per command (closed in `finally`). Thread-safe via `_wss_lock`. An earlier persistent-connection design was abandoned after Indigo 2025.2 / Python 3.13 / `websockets` 15.x began raising `ProtocolError("incorrect masking")` on long-lived sockets — the per-call model trades one TLS handshake per poll for immunity to that class of state-desync bug.
- **Timeout**: 8 seconds for connection, 10 seconds for response

**Legacy TCP (port 4242):**
- **Port**: 4242 (unencrypted, no auth)
- **Protocol**: JSON over TCP with null terminator (`\0`). Opens a new socket per command.
- **Timeout**: 8 seconds for socket operations

- **IP Address**: Configurable via plugin preferences (default: 192.168.0.1)

All commands are sent as JSON objects: `{"COMMAND":value}`. The WSS path wraps them in the token-authenticated envelope; the TCP path sends them directly with a null terminator.

### Concurrent Thread Architecture

The plugin runs a continuous monitoring thread ([plugin.py:84-108](HeatmiserNeo.IndigoPlugin/Contents/Server Plugin/plugin.py#L84-L108)) that:

1. **Every 30 seconds**: Polls Neohub for device status updates via `INFO` command
2. **Once daily at specific times**:
   - 03:00 - Synchronizes Neohub time/date with Indigo (if enabled)
   - 00:00 - Fetches DCB (Device Control Block) data
   - 05:00 - Fetches Engineering data
3. **On first run**: Configures NTP settings based on time sync preference

**Thread Sleep Pattern**: 1 second poll + 29 second wait = 30 second cycle

### Device Auto-Discovery

On startup, the plugin queries the Neohub for all connected devices ([plugin.py:144-186](HeatmiserNeo.IndigoPlugin/Contents/Server Plugin/plugin.py#L144-L186)):

1. Sends `{"INFO":0}` command to discover devices
2. Creates Indigo device for each physical device (if not already created)
3. Uses the device's array index in the Neohub as the Indigo device address
4. Automatically determines device type and creates appropriate Indigo device

**Device Naming Restriction**: Device names from Heatmiser must be valid Python identifiers (no spaces, special characters except underscore). The plugin validates this with `isidentifier()` and logs an error if invalid.

### Supported Device Types

| Device Type | Description | Indigo Device Type |
|-------------|-------------|-------------------|
| 1 | NeoStat thermostat | `heatmiserNeostat` |
| 6 | NeoPlug smart plug | `heatmiserNeoplug` |
| 7 | NeoAir thermostat | `heatmiserNeostat` |
| 12 | NeoStat-e thermostat | `heatmiserNeostat` |
| 13 | NeoAir (newer model) | `heatmiserNeostat` |
| 14 | Wireless air sensor | `heatmiserNeoSensor` |
| 0 | Offline device | (sets error state) |

### State Management

**NeoStat Thermostats** track:
- Current temperature, setpoint, heat state
- Pre-heat mode, frost protection (mapped to "Cool" mode)
- Away/Holiday modes
- Hold time remaining and hold temperature
- Window open, low battery, keypad locked
- Floor temperature (when probe present, 127 = no probe)
- Hub firmware version, DST/NTP status (stored on first device)
- Engineering data: Rate of Change, Frost Temp, Switching Differential

**NeoTimeclock** tracks:
- On/Off state, temperature, short mode
- Away/Holiday modes
- Hold time remaining (for timer boost)

**NeoPlug** tracks:
- On/Off state based on timer status

**Wireless Air Sensors** track:
- Current temperature reading
- Sensor valid/online status
- Low battery warning

### Mode Mapping

Heatmiser modes are mapped to Indigo HVAC modes:

| Heatmiser Mode | Indigo Mode | Description |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [simons-plugins/heatmiser](https://github.com/simons-plugins/heatmiser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
