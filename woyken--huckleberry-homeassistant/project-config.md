---
trigger: always_on
description: This is a **Home Assistant custom integration** for the Huckleberry baby tracking app. It provides real-time sleep, feeding, diaper, and growth tracking by using the [`huckleberry-api`](https://pypi.org/project/huckleberry-api/) Python library to connect to Huckleberry's Firebase backend.
---

# AI Agent Context Guide - Huckleberry Home Assistant Integration

## Project Overview

This is a **Home Assistant custom integration** for the Huckleberry baby tracking app. It provides real-time sleep, feeding, diaper, and growth tracking by using the [`huckleberry-api`](https://pypi.org/project/huckleberry-api/) Python library to connect to Huckleberry's Firebase backend.

**Critical Context**: This integration wraps the `huckleberry-api` library and adapts it for Home Assistant's architecture. The API handles all Firebase operations, while this integration provides HA-specific features like entities, services, automations, and real-time updates through coordinators.

## Related Documentation

- **[Project Overview](../AGENTS.md)** - High-level context, history, decompiled source analysis
- **[API Library](../huckleberry-api/AGENTS.md)** - Underlying Firebase operations and data structures

## Critical Agent Rule

When adding or changing any enum value, state value, mode, unit, key name, option list, or any other value that originates from the Huckleberry app or Firebase schema, you **must** validate the value against API and make sure to find evidence that it exists first.

Never add guessed, inferred, placeholder, or convenience values.

If a value cannot be validated as factual and true in API evidence, do not add it.

When changing dependencies for this integration, always update both `custom_components/huckleberry/manifest.json` and `pyproject.toml` so metadata and local development dependencies stay in sync.

For Python commands in this repository (including running tests), always use the `uv` CLI (for example: `uv run pytest ...`) instead of invoking tools directly.

## Project Purpose

This integration provides:
- Home Assistant entities for baby tracking (sensors, switches)
- Services for controlling tracking from automations
- Device actions for advanced automation scenarios
- Real-time updates via coordinator and Firebase listeners
- Multi-child support with device grouping
- HACS installation support

## Architecture

### Technology Stack

**Home Assistant:**
- Custom Component (local integration)
- Config Flow for UI configuration
- Data Update Coordinator pattern
- Async/await architecture
- Real-time entity updates

**Platforms:**
- `switch`: Sleep timer + left/right nursing switches per child
- `sensor`: Sleep/nursing status + Children count + child profile + growth sensors

**External Dependencies:**
- `huckleberry-api>=0.2.2` - Firebase operations

### Integration Structure

Platform files should stay thin. Keep `sensor.py`, `switch.py`, and other Home Assistant platform modules focused on `async_setup_entry()` and entity assembly, while entity implementations live in feature modules grouped by Huckleberry domain.

```
custom_components/huckleberry/
├── __init__.py              # Integration setup, coordinator, service registration
├── api.py                   # Legacy API (deprecated - will be removed)
├── config_flow.py           # Configuration UI flow
├── const.py                 # Constants (DOMAIN, PLATFORMS)
├── switch.py                # Switch platform entrypoint; assembles feature switches
├── sensor.py                # Sensor platform entrypoint; assembles feature sensors
├── features/                # Feature-oriented entity modules grouped by Huckleberry domain
│   ├── child.py             # Children and child profile sensors
│   ├── sleep.py             # Sleep sensor and sleep switch
│   ├── nursing.py           # Nursing sensor and nursing switches
│   ├── bottle.py            # Bottle sensor
│   ├── diaper.py            # Diaper sensor
│   └── growth.py            # Growth sensor
├── services.yaml            # Service definitions with device selectors
├── manifest.json            # Integration metadata, dependencies
├── strings.json             # UI strings for config flow
├── translations/            # Localization files
│   └── en.json
├── README.md                # User installation guide
└── AGENTS.md                # This file
```

### Key Classes

**`HuckleberryDataUpdateCoordinator`** (`__init__.py`):
- Extends `DataUpdateCoordinator` from Home Assistant
- Manages data updates for all entities
- Sets up real-time Firebase listeners via `async_setup_listeners()`
- Stores data in `_realtime_data` dict by child_uid
- Structure: `{child_uid: {"sleep_status": {...}, "feed_status": {...}, "health_status": {...}}}`
- Provides `get_sleep_status()`, `get_feed_status()`, `get_health_status()` helper methods
- Handles listener cleanup on unload

**Service Implementation Pattern**:

Services use device selector for user convenience:

1. **services.yaml**: Define `device_id` field with `selector: device: integration: huckleberry`
2. **__init__.py**: `_get_child_uid_from_call()` extracts child_uid from device_id
3. **Priority**: Explicit `child_uid` > device_id > first child fallback

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Woyken/huckleberry-homeassistant](https://github.com/Woyken/huckleberry-homeassistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
