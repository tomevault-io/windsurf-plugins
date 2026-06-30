---
trigger: always_on
description: This is a **Home Assistant custom component** for monitoring air quality data from the City of Montreal's official monitoring stations. It provides real-time AQI (Air Quality Index) values, qualitative air quality levels, and detailed pollutant concentration sensors using the official Montreal open data API.
---

# Montreal AQI Home Assistant Integration - AI Coding Agent Guide

## Project Overview

This is a **Home Assistant custom component** for monitoring air quality data from the City of Montreal's official monitoring stations. It provides real-time AQI (Air Quality Index) values, qualitative air quality levels, and detailed pollutant concentration sensors using the official Montreal open data API.

**Tech Stack**: Python 3.12+, Home Assistant 2024.12.0+, async/await, `montreal-aqi-api` library

## Architecture

### Core Components

1. **`coordinator.py`** - `MontrealAQICoordinator`: Central data fetcher using HA's `DataUpdateCoordinator`
   - Fetches data from Montreal's RSQA (Réseau de surveillance de la qualité de l'air) API
   - Updates every 30 minutes (configurable via `UPDATE_INTERVAL` in `const.py`)
   - Handles API errors gracefully with `UpdateFailed` exceptions
   - Returns structured data: `aqi`, `dominant_pollutant`, `pollutants` dict, and `timestamp`

2. **`api.py`** - `MontrealAQIApi`: Async wrapper for the `montreal-aqi-api` library
   - Provides async methods: `async_list_stations()` and `async_get_station()`
   - Uses `hass.async_add_executor_job()` to run synchronous API calls in thread pool
   - Handles station listing for config flow and data fetching for coordinator

3. **`config_flow.py`** - `MontrealAQIConfigFlow`: Multi-step config flow for station selection
   - Single step: Lists available monitoring stations and lets user select one
   - Fetches station list from API during setup
   - Creates config entry with `station_id`

4. **`sensor.py`** - Entity implementations
   - **AQI Index Sensor**: Numeric AQI value (0-100+)
   - **AQI Level Sensor**: Qualitative level ("Good", "Acceptable", "Bad") with dominant pollutant attribute
   - **Pollutant Sensors**: Individual sensors for PM2.5, PM10, NO₂, O₃, SO₂, CO concentrations
   - **Timestamp Sensor**: Last measurement timestamp
   - All sensors inherit from `CoordinatorEntity` and share device info

5. **`const.py`** - Single source of truth for constants and sensor definitions
   - Sensor entity descriptions (`AQI_DESCRIPTION`, `AQI_LEVEL_DESCRIPTION`)
   - Pollutant mappings (`DEVICE_CLASS_MAP`) with device classes, units, icons
   - Update interval and configuration keys
   - Unit conversions (PPB to µg/m³)

### Data Flow

1. **Config Flow**: User selects station → Config entry created with `station_id`
2. **Setup**: Coordinator created with API instance → First data fetch
3. **Polling**: Every 30 minutes, coordinator calls `api.async_get_station(station_id)`
4. **Data Processing**: API returns dict with AQI, pollutants, timestamp
5. **Entity Updates**: All sensors update from coordinator data

### Sensor Types

| Sensor Type | Description | Device Class |
|-------------|-------------|--------------|
| AQI Index | Numeric AQI value | `SensorDeviceClass.AQI` |
| AQI Level | Qualitative level (Good/Acceptable/Bad) | `SensorDeviceClass.ENUM` |
| Pollutants | Individual pollutant concentrations | Varies (PM25, PM10, NO2, etc.) |
| Timestamp | Last measurement time | `SensorDeviceClass.TIMESTAMP` |

**Pollutant Sensors**: Only created for pollutants present in API response. Concentrations are in µg/m³.

## Setup (use devcontainer for consistency)

This project uses standard Python packaging with pip and modern tools for performance.

```bash
# Install dependencies with uv (faster than pip)
uv pip install -r requirements.txt
uv pip install -r requirements_dev.txt

# For testing
uv pip install -r requirements_test.txt
```

```bash
# Linting and formatting
ruff check custom_components/
ruff format custom_components/

# Type checking
mypy custom_components/montreal_aqi/

# Testing
pytest -v

# Coverage
pytest --cov=custom_components.montreal_aqi --cov-report=html

# Pre-commit hooks with prek (faster than pre-commit)
prek install
prek run --all-files
```

### Key Files to Check First

- `README.md`: User-facing features, installation, entity descriptions
- `DOCUMENTATION.md`: Technical details, data source, dashboard examples
- `pyproject.toml`: Python version, pytest/ruff/mypy config
- `requirements.txt*`: Dependencies for runtime, dev, and testing
- `custom_components/montreal_aqi/const.py`: Sensor definitions and constants
- `tests/conftest.py`: Test fixtures and mocks

## Dependency Management

### Key Dependencies

- `montreal-aqi-api==0.5.0` - Official Montreal AQI API wrapper (pinned in `manifest.json`)
- `homeassistant>=2024.12.0` - For development and testing
- `pytest-homeassistant-custom-component` - HA-specific testing utilities

### Updating Dependencies

- Update version in `manifest.json` and `requirements.txt`
- Test compatibility with HA version matrix
- Check API changes in `montreal-aqi-api` library

## Code Conventions

### Type Hints & Style

- Strict `mypy`: All functions need type hints
- Imports: Standard library → Third-party → HA → Local
- Line length: 88 chars (ruff default)
- Logging: Use `_LOGGER.debug/info/warning/error` with context


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [normcyr/home-assistant-montreal-aqi](https://github.com/normcyr/home-assistant-montreal-aqi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
