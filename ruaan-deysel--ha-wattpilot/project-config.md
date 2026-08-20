---
trigger: always_on
description: Custom Home Assistant integration for Fronius Wattpilot EV chargers using wattpilot-api (WebSocket gateway to chargers on local LAN or Fronius Cloud).
---

# Wattpilot-HA Copilot Instructions

Custom Home Assistant integration for Fronius Wattpilot EV chargers using wattpilot-api (WebSocket gateway to chargers on local LAN or Fronius Cloud).

## Quick Start

```bash
scripts/setup      # Install uv, sync deps, install pre-commit hooks
scripts/develop    # Start HA at http://localhost:8123 with debug logging
scripts/lint       # Format and lint (ruff format + check --fix) - REQUIRED before commits
```

Dependency management uses `uv` with `pyproject.toml` + `uv.lock`. Config in `config/`. Debug logging already enabled in `config/configuration.yaml`.

## Architecture Overview

**Data Flow**: `WebSocket (wattpilot-api) → charger.all_properties → WattpilotCoordinator → CoordinatorEntity → HA state`

| Component         | Purpose                                                                                    |
| ----------------- | ------------------------------------------------------------------------------------------ |
| `__init__.py`     | Entry point: connects charger, creates coordinator, registers services, forwards platforms |
| `coordinator.py`  | `WattpilotCoordinator` wraps charger updates, provides centralized availability state      |
| `entities.py`     | Base `ChargerPlatformEntity(CoordinatorEntity)` with firmware/variant/connection filters   |
| `descriptions.py` | Dataclass entity descriptions (replaces YAML per-platform configs)                         |
| `{platform}.py`   | Platform implementation that loads descriptions and creates entities                       |
| `utils.py`        | Property helpers (`GetChargerProp`, `async_GetChargerProp`, `async_SetChargerProp`)        |
| `services.py`     | Integration services (set_next_trip, disconnect_charger, etc.)                             |

## wattpilot-api Package

The integration uses the external `wattpilot-api>=1.1.0` package (from PyPI). Key types and patterns:

```python
from wattpilot_api import Wattpilot, LoadMode, CarStatus

# Main client class with WebSocket connection
charger = Wattpilot(host, password)  # local connection
await charger.connect()
await charger.disconnect()

# Properties access
charger.all_properties  # Dict of all current property values
charger.connected  # Boolean connection state
charger.properties_initialized  # Boolean - all properties loaded
charger.name, charger.serial, charger.firmware  # Device identifiers

# Callbacks (async-friendly)
async def on_property_change(identifier: str, value: Any) -> None:
    ...

unsub = charger.on_property_change(on_property_change)  # returns unsubscribe callable
unsub()  # disconnect the callback

# Enums and constants
LoadMode.DEFAULT  # 3
LoadMode.ECO      # 4
LoadMode.NEXTTRIP # 5
```

## Entity Descriptions (descriptions.py)

Entities are defined as dataclass descriptions in `descriptions.py`, not separate YAML files. Pattern:

```python
from homeassistant.components.sensor import SensorEntityDescription
from .descriptions import (
    WattpilotSensorEntityDescription,
    SENSOR_DESCRIPTIONS,
    SOURCE_PROPERTY,
    filter_descriptions
)

# In platform setup
descriptions = filter_descriptions(SENSOR_DESCRIPTIONS, charger, entry, charger_id)
for desc in descriptions:
    entity = ChargerSensor(hass, entry, desc, charger)
    entities.append(entity)

# Description definition (in descriptions.py)
WattpilotSensorEntityDescription(
    key="session_energy",
    charger_key="wh",  # go-eCharger API key
    name="Session Energy",
    source=SOURCE_PROPERTY,  # "property" | "attribute" | "namespacelist"
    device_class=SensorDeviceClass.ENERGY,
    state_class=SensorStateClass.TOTAL_INCREASING,
    native_unit_of_measurement="Wh",
    firmware=">=38.5",  # Optional version constraint
    variant="11",        # Optional variant filter
    connection="local",  # Optional connection filter
)
```

**Adding entities**:

1. Find charger API key in [go-eCharger API v2](https://github.com/goecharger/go-eCharger-API-v2/blob/main/apikeys-en.md)
2. Create `WattpilotXXXEntityDescription` in `descriptions.py` with appropriate `source` and platform-specific fields
3. Add to `XXX_DESCRIPTIONS` list - entity auto-created by platform setup

## Data Update Coordinator Pattern

The integration uses `DataUpdateCoordinator` for centralized data management and availability handling:

```python
# In __init__.py during setup_entry
coordinator = WattpilotCoordinator(hass, charger, entry)
await coordinator.async_config_entry_first_refresh()

entry.runtime_data = WattpilotRuntimeData(
    charger=charger,
    coordinator=coordinator,  # Coordinator instance
    push_entities={},
    params=dict(entry.data),
)

# Register callback for WebSocket property updates
async def _on_property_change(identifier: str, value: Any) -> None:
    await async_property_update_handler(hass, entry, identifier, value)

unsub = charger.on_property_change(_on_property_change)
entry.runtime_data.property_updates_callback = unsub  # Save for cleanup
```

**Coordinator state management**:

- `coordinator.data`: Current `charger.all_properties` dict
- `coordinator.available`: True if charger connected AND properties initialized

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ruaan-deysel/ha-wattpilot](https://github.com/ruaan-deysel/ha-wattpilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
