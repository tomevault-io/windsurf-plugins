---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Home Assistant custom integration for Fairland pool heat pumps and pool pumps (Inverflow Plus and OEM rebrands such as Madimack). It connects directly to Fairland's iGarden cloud API (not Tuya) to monitor and control devices.

## Development Commands

```bash
# Install dependencies
scripts/setup

# Run linting (ruff format + ruff check with auto-fix)
scripts/lint

# Run the test suite (no HA install needed; see "Verifying Against Real Devices")
pip install -r requirements-test.txt && pytest tests/
# or, without setup: uvx --python 3.13 pytest tests/

# Run local Home Assistant instance for testing
scripts/develop
```

`scripts/lint` needs `ruff` on PATH; if it is missing, `uvx ruff format custom_components/ && uvx ruff check --fix custom_components/` works without setup. Note CI lints the whole repo (`ruff check .`), so `tests/` must lint clean too.

The `scripts/develop` command creates a `config/` directory and starts Home Assistant with debug logging enabled. The custom component is added to PYTHONPATH automatically.

## Architecture

### Data Flow

1. **FairlandApiClient** (`api.py`) - HTTP client for the iGarden cloud
   - Four regional servers (`API_REGIONS` in `const.py`); an account exists on exactly ONE region, auto-detected by trying each at login (#74)
   - Handles authentication (login with password, token management)
   - Fetches courtyards (device groups) and devices
   - Gets/sets device data points (dps)
   - The cloud allows only one active session per account (server-side, #69)

2. **FairlandDataUpdateCoordinator** (`coordinator.py`) - Polls API at configurable interval (default 30s)
   - Fetches all devices in selected courtyard
   - Updates device status data points for each device

3. **FairlandData** (`data.py`) - Runtime data container attached to config entry
   - Holds client, coordinator, and integration references

4. **Entity Platforms** - Binary Sensor, Climate, Sensor, Switch, Number, Select entities created per device, dispatched on `categoryCode` (`heatPump`, `waterPump`, `saltMachine`, `sandCylinder`).

### Device Data Points (dps)

Devices expose functionality through numbered data points. **The dpId namespace is per `categoryCode`** — e.g. heat-pump dp 108 = Lower Temperature Limit, water-pump dp 108 = Backwash Countdown. Never share dp maps across categories.

Heat pump (`heatPump`): `101` power switch, `102` running/preset mode, `103` current temperature, `106` HVAC mode (0=Auto, 1=Heat, 2=Cool), `107` target temperature, `113` operating status, `138` remote-switch status (enum 0=on/1=off — exposed as an enum sensor, not a bool, so a 0 value isn't read as "off").

Water pump (`waterPump`): `5` current power, `102` running rate, `103` mode enum, `104` backwash duration, `105` power switch, `108` backwash countdown, `109` energy, `111` speed setpoint.

Salt chlorinator (`saltMachine`, #80): `101` salt ppm, `103` power, `108` ORP setpoint, `110` pH setpoint (scale 1), `111`/`112` ORP/pH measured, `102`/`105`/`133` temps (102=pool °C, 133=pool °F, 105=controller), `119` water-quality enum, `122` polarity-reversal interval, `125`/`113` chlorine output target/actual, `132` mode enum, `114`/`115`/`117`/`118`/`154` binary status/alarms. Binary alarm polarity is derived from the dpProperty true/false labels, not guessed.

Multiport valve (`sandCylinder`, #80/#81): `101` water temp (scale 1), `102` pressure (MPa, scale 3 — HA has no MPa unit so it rides as a plain unit string), `105` backwash-trigger pressure (MPa setpoint), `106` mode-switch command enum, `107` current valve position enum, `108`/`109`/`111`/`112`/`113` backwash/protection setpoints, `115`/`116` countdown sensors, `118`/`123`/`125` config/pump selects. Entity names and enum option names are taken verbatim from the firmware's own `nameLanguage`/`propNameLanguage` (en-US) in the diagnostics. Enum labels here are localized (Chinese), so `sandCylinder` selects map by the integer key (`int_to_option`) instead of by label.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [siedi/ha-fairland](https://github.com/siedi/ha-fairland) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-01 -->
