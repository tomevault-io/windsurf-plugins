---
trigger: always_on
description: Home Assistant custom component for Bosch HomeCom Easy-connected appliances. Cloud-polling integration (`iot_class: cloud_polling`) using the `homecom_alt` library for API communication via Bosch SingleKey ID OAuth2 authentication.
---

# CLAUDE.md - Bosch HomeCom Home Assistant Integration

## Project Overview

Home Assistant custom component for Bosch HomeCom Easy-connected appliances. Cloud-polling integration (`iot_class: cloud_polling`) using the `homecom_alt` library for API communication via Bosch SingleKey ID OAuth2 authentication.

**Domain:** `bosch_homecom`
**Source:** `custom_components/bosch_homecom/`

## Supported Device Types

| Type | Model | Coordinator | API Client |
|------|-------|-------------|------------|
| `rac` | Residential Air Conditioning | `BoschComModuleCoordinatorRac` | `HomeComRac` |
| `k30`, `k40` | Bosch boiler | `BoschComModuleCoordinatorK40` | `HomeComK40` |
| `icom` | Bosch Heat Pump | `BoschComModuleCoordinatorK40` | `HomeComK40` |
| `rrc2` | Bosch Thermostat | `BoschComModuleCoordinatorK40` | `HomeComK40` |
| `wddw2` | Bosch Water Heater | `BoschComModuleCoordinatorWddw2` | `HomeComWddw2` |

## Architecture

### Data Flow

1. `config_flow.py` — Multi-step auth: username -> browser OAuth code -> device selection
2. `__init__.py` — Creates one coordinator per selected device, registers custom services
3. `coordinator.py` — `DataUpdateCoordinator` subclasses poll the API (default 60s, configurable 15-3600s via options flow)
4. Entity files — `CoordinatorEntity` subclasses read from `coordinator.data` and write via `coordinator.bhc`

### Platforms

Six entity platforms in `PLATFORMS`: `climate`, `fan`, `select`, `sensor`, `switch`, `water_heater`.

Each platform's `async_setup_entry` iterates coordinators, checks `deviceType`, and creates entities accordingly.

### Coordinator Pattern

All four coordinator classes (`Generic`, `Rac`, `K40`, `Wddw2`) in `coordinator.py` follow identical structure:
- Inherit `DataUpdateCoordinator[BHCDevice*]`
- Store `self.bhc` (API client), `self.unique_id`, `self.device`, `self.entry`, `self.auth_provider`
- `_async_update_data()` refreshes tokens (if `auth_provider`), calls `self.bhc.async_update()`, returns typed data object
- Token refresh persists to config entry via `hass.config_entries.async_update_entry()`
- Catches `ApiError`, `InvalidSensorDataError`, `RetryError` -> `UpdateFailed`; `AuthFailedError` triggers reauth

### Entity Pattern

All entities:
- Inherit `CoordinatorEntity` + platform entity (e.g., `ClimateEntity`)
- Set `_attr_has_entity_name = True`, `_attr_should_poll = False`
- Use `_attr_unique_id = f"{coordinator.unique_id}-{field}-{suffix}"`
- Use `_attr_translation_key` for names
- Implement `_handle_coordinator_update()` -> `self.set_attr()` + `self.async_write_ha_state()`
- Control methods call `self.coordinator.bhc.async_*()` then `self.coordinator.async_request_refresh()`

### Services (registered in `async_setup`)

- `set_dhw_tempreture` — Set DHW temperature for a level
- `set_dhw_extrahot_water` — Control extra hot water charge mode/duration
- `get_custom_path_service` — Query arbitrary API paths (returns response)

## Key Files

| File | Purpose |
|------|---------|
| `__init__.py` | Entry point: coordinator creation, service registration, options listener |
| `coordinator.py` | 4 coordinator classes (Generic, Rac, K40, Wddw2) |
| `config_flow.py` | ConfigFlow (user/browser/devices steps), OptionsFlow, reauth/reconfigure |
| `const.py` | Constants, device model map, attribute names, sensor descriptors, login URL |
| `climate.py` | `BoschComRacClimate` (AC), `BoschComK40Climate` (heating circuits) |
| `sensor.py` | Notification, DHW, HC, heat source, ventilation, outdoor temp sensors |
| `select.py` | Airflow, program, DHW mode, HC mode, away/holiday selects |
| `switch.py` | Plasmacluster air purification (RAC only) |
| `fan.py` | Ventilation fan (K30/K40/ICOM) |
| `water_heater.py` | K40 and WDDW2 water heater entities |
| `diagnostics.py` | Config entry diagnostics |
| `strings.json` | UI strings (config flow, options, errors, entity translations) |

## Code Conventions

- `from __future__ import annotations` in all files
- Type hints use Python 3.10+ union syntax (`str | None`)
- Async throughout; all HA-facing methods are `async def`
- `@callback` decorator on synchronous coordinator update handlers
- `_LOGGER = logging.getLogger(__name__)` per module
- `PARALLEL_UPDATES = 1` in entity platform files
- Private attributes prefixed with `_` (e.g., `self._operationMode`)
- Data extraction uses `entry.get("key", {}).get("value")` pattern with nested dicts from API
- Import order: `__future__` -> stdlib -> third-party (`homecom_alt`, `homeassistant`) -> local (`.const`, `.coordinator`)

## Build and Test

### Virtual Environment (REQUIRED)

**Always use the project venv for all commands.** Create it once, then activate before running anything:

```bash
# Create venv (first time only)
python3 -m venv venv

# Activate venv (every session)
source venv/bin/activate

# Install dependencies
pip install -r requirements_test.txt
# or if no requirements file:
pip install pytest pytest-cov pytest-homeassistant-custom-component coverage ruff black isort homecom_alt tenacity
```

### Dependencies

- Runtime: `homecom_alt>=1.5.3`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [serbanb11/bosch-homecom-hass](https://github.com/serbanb11/bosch-homecom-hass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
