---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Home Assistant custom integration for Ecowitt weather stations that uses local web interface polling instead of webhooks. The integration creates stable entity IDs based on hardware IDs and organizes sensors into individual devices for better management.

## ⚠️ CRITICAL: Entity Creation Pipeline Issues

**IMPORTANT**: This codebase has ongoing entity creation failures despite proper device detection. The primary issues are incomplete entity sets and mapping pipeline failures, not architectural violations. Focus on debugging the entity creation flow rather than mapping logic.

## Development Commands

### Local environment
The project uses a virtualenv at `.venv/` in the project root.
- **Python**: Homebrew Python 3.14 at `/opt/homebrew/bin/python3`
- **Create venv**: `python3 -m venv .venv && .venv/bin/pip install -r requirements_test.txt`

### Testing
```bash
# Run all tests with coverage (ALWAYS use .venv, not system python)
PYTHONPATH="$PWD" .venv/bin/pytest tests/ -v

# Run with coverage report
PYTHONPATH="$PWD" .venv/bin/pytest tests/ --cov=custom_components/ecowitt_local --cov-report=term-missing
```

**Coverage must be 100%.** If coverage drops, add tests before committing.

### ⚠️ Mandatory pre-commit checklist

Run ALL of these before every commit. CI will fail if any step is skipped.

```bash
# 1. Format
black custom_components/ecowitt_local/ tests/

# 2. Sort imports
isort custom_components/ecowitt_local/ tests/

# 3. Type check (must show "Success: no issues found")
mypy custom_components/ecowitt_local/ --follow-imports=silent --ignore-missing-imports

# 4. Full test suite with coverage (must be 100%)
PYTHONPATH="$PWD" .venv/bin/pytest tests/ --cov=custom_components/ecowitt_local --cov-report=term-missing
```

### Development Dependencies
Install test requirements: `pip install -r requirements_test.txt`

## Architecture Overview

### Core Components

**Integration Entry Point** (`__init__.py:23`): Sets up coordinator, device registry, platforms, and services in a specific order to ensure proper entity-device relationships.

**Data Coordinator** (`coordinator.py`): Manages two separate polling intervals:
- Live data polling (60s default) for sensor readings
- Sensor mapping polling (600s default) for hardware ID discovery

**API Client** (`api.py`): Handles authentication and communication with Ecowitt gateway web interface. Supports both authenticated and non-authenticated gateways.

**Sensor Mapping** (`sensor_mapper.py`): Critical component that maps raw sensor data keys to hardware IDs and device information. This enables stable entity IDs and proper device organization.

### Entity Organization

The integration creates individual Home Assistant devices for each physical sensor rather than grouping everything under the gateway. Each device contains:
- Primary sensor entities (temperature, humidity, etc.)
- Diagnostic entities (battery, signal strength, online status)

### Hardware ID Strategy

Entity IDs are based on hardware IDs extracted from the gateway's sensor mapping, ensuring stability across battery changes and reconnections:
- `sensor.ecowitt_soil_moisture_d8174` (hardware ID: D8174)
- `sensor.ecowitt_temperature_a1b2c3` (hardware ID: A1B2C3)

## Key Integration Points

### Device Registry Setup
Device creation happens in `__init__.py:40` before platform setup to ensure entities find their proper parent devices.

### Migration System
The integration includes migration logic in `__init__.py` to handle updates from older versions and reassign entities to correct devices.

### Error Handling
Authentication errors trigger re-authentication. API failures are logged but don't crash the integration, allowing recovery on next update cycle.

## Testing Strategy

The codebase uses comprehensive mock data testing for device types not physically available, validated against the `aioecowitt` library's sensor mappings. Physical testing is done with WH51 soil moisture sensors.

Test coverage is maintained at **100%** with 330+ automated tests covering device discovery, entity creation, hardware ID mapping, and edge cases.

## Configuration

The integration uses Home Assistant's config flow with these key options:
- Host IP address (required)
- Password (optional, for protected gateways)
- Live data polling interval (30-300s)
- Sensor mapping interval (300-3600s)
- Include inactive sensors flag

---

# 🎯 Issue Analysis Patterns & Solutions

*The following patterns were learned through extensive issue analysis. These guide how to approach common problems in this integration.*

## Common Device Mapping Issues

### Pattern: Device Type String Mismatch
- **Problem**: New weather stations report device type strings that don't match expected patterns
- **Example**: WH90 reports as `"Temp & Humidity & Solar & Wind & Rain"` instead of `"wh90"`
- **Solution**: Add device type string matching to `sensor_mapper.py` 
- **Files**: `custom_components/ecowitt_local/sensor_mapper.py`
- **Approach**: Extend existing `elif` conditions with `or "actual device string" in sensor_type.lower()`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alexlenk/ecowitt_local](https://github.com/alexlenk/ecowitt_local) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
