---
trigger: always_on
description: This document provides guidance for AI coding agents working on this Home Assistant custom integration project.
---

# AI Agent Instructions - Air Sviva Custom Component

This document provides guidance for AI coding agents working on this Home Assistant custom integration project.

## Project Overview

This is a Home Assistant custom component for the Israeli Ministry of Environmental Protection air quality API ([air.sviva.gov.il](https://air.sviva.gov.il)). It provides sensors for air quality monitoring data.

**Integration details:**
- **Domain:** `air_sviva`
- **Title:** Air Sviva
- **Repository:** GuyKh/air-sviva-custom-component

**Key directories:**
- `custom_components/air_sviva/` - Main integration code
- `.github/workflows/` - CI/CD workflows

## Tech Stack

- **Python**: 3.12+
- **Home Assistant**: 2023.0+
- **Air Sviva API**: 0.0.1 (Python client for air.sviva.gov.il API)
- **Linting**: ruff (with Home Assistant rules)
- **Type Checking**: mypy (strict mode)

## Code Structure

```
custom_components/air_sviva/
├── __init__.py          # Integration entry point (async_setup_entry, async_unload_entry)
├── const.py             # Constants and DOMAIN definitions
├── config_flow.py       # UI configuration flow (region + station selection)
├── coordinator.py       # DataUpdateCoordinator - API polling
├── data.py              # Runtime data store
├── entity.py            # Base coordinator entity
├── sensor.py            # Sensor platform (main entity logic)
├── manifest.json        # Integration manifest
├── strings.json         # Config flow strings (English)
└── translations/
    ├── en.json          # English translations (sensor names, config flow)
    └── he.json          # Hebrew translations (sensor names, config flow)
```

## Local Development

**Always use project scripts** — do NOT craft your own `hass`, `pip`, or similar commands. The scripts handle environment setup correctly.

**Setup:**
```bash
./scripts/setup  # Install dependencies
```

**Start Home Assistant:**
```bash
./scripts/develop  # Start HA development environment
```

**Debugging:**
Enable debug logging in `config/configuration.yaml`:
```yaml
logger:
  default: info
  logs:
    custom_components.air_sviva: debug
    air_sviva_api: debug
```

**Reading logs:**
- Terminal where `./scripts/develop` runs
- `config/home-assistant.log`

## Workflow

### Starting New Work

**When starting a new task, always ask the user first:**
- Should I switch to `main` branch and rebase?
- Or should I work from the current branch?

Then checkout a new feature branch before beginning work. Never work directly on `main` or stale branches.

### Branch Naming Convention
- Features: `feat/description`
- Bug fixes: `fix/description`
- Documentation: `docs/description`

## Code Style

**Python:**
- 4 spaces indentation
- 120 character lines
- Double quotes for strings
- Full type hints (mypy strict)
- Async for all I/O operations
- Follow ruff rules from `.ruff.toml`

**Validation commands:**
```bash
ruff check custom_components/air_sviva/
ruff format custom_components/air_sviva/ --check
mypy custom_components/air_sviva/
```

## Key Patterns

### Integration Setup
- Uses `ConfigEntry` for UI-based configuration
- Supports one entry per station
- Registers `PLATFORMS`: `sensor`
- Stores data in `hass.data[DOMAIN][entry.entry_id]`

### Coordinator Pattern
- `AirSvivaUpdateCoordinator` extends `DataUpdateCoordinator`
- Fetches latest data from all regions, finds the configured station
- Parses channel data into `{channel_name: channel_data}` dict
- Entities read from `coordinator.data`, never call API directly
- Raise `ConfigEntryAuthFailed` (triggers reauth) or `UpdateFailed` (retry)

### Entity Pattern
- Base `AirSvivaEntity` in `entity.py` — extends `CoordinatorEntity`
- `AirSvivaSensor` in `sensor.py` — main sensor entity
- Uses `_attr_translation_key` for per-language display names
- Entity ID format: `sensor.sviva_station_{station_id}_{pollutant}`
- Unique ID format: `sviva_station_{station_id}_{pollutant}`
- Wind direction sensors (WD, WDD) get `is_circular: True`, `min_value: 0`, `max_value: 360`

### Config Flow
- Implement in `config_flow.py`
- Step 1: Select region (`_async_fetch_regions`)
- Step 2: Select station with auto-proximity (`_async_fetch_stations`)
- Sets `unique_id` for entries

### Translation System
- Sensor names use HA's `_attr_translation_key` system
- Translation keys are short identifiers (e.g., `"so2"`, `"temp"`, `"wds"`)
- HA constructs the full path: `component.air_sviva.entity.sensor.{key}.name`
- `en.json` has English names, `he.json` has Hebrew names for non-scientific terms
- Scientific notation (SO₂, PM2.5, NO₂, etc.) is identical in both languages
- Adding a new sensor type requires entries in ALL translation files

## Project-Specific Rules

### Air Sviva Identifiers
- **Domain:** `air_sviva`
- **Class prefix:** `AirSviva`

### Data Model
- `channels` dict keyed by pollutant name (e.g., "SO₂", "WD", "TEMP")
- Each channel has: `id`, `name`, `value`, `units`, `pollutant_id`, `alias`, `description`, `datetime`
- Pollutant names from API are normalized for translation keys (lowercase, underscores)

### Pollutant Naming Rules
- **Scientific/chemical notation** (SO₂, NO₂, PM2.5, O₃, CO, CH₄, etc.) must NEVER be translated — they appear identical in all language files

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GuyKh/air-sviva-custom-component](https://github.com/GuyKh/air-sviva-custom-component) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
