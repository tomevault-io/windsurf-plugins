---
trigger: always_on
description: This is a custom Home Assistant integration for Xenia Espresso Machines. These instructions help ensure code quality and consistency.
---

# Claude Code Instructions - Xenia Espresso Machine Integration

This is a custom Home Assistant integration for Xenia Espresso Machines. These instructions help ensure code quality and consistency.

## Project Overview

- **Domain**: `xenia_home`
- **Integration Type**: Device (local polling)
- **IoT Class**: Local polling
- **Platforms**: sensor, binary_sensor, number, select, switch, button, event
- **Codeowners**: @knoedelauflauf

## Python Requirements

- **Compatibility**: Python 3.13+
- **Language Features**: Use the newest features when possible:
  - Pattern matching
  - Type hints
  - f-strings (preferred over `%` or `.format()`)
  - Dataclasses
  - Walrus operator

## Code Quality Standards

- **Formatting**: Ruff
- **Linting**: Ruff
- **Type Checking**: MyPy
- **Testing**: pytest with plain functions and fixtures
- **Language**: American English for all code, comments, and documentation (use sentence case, including titles)

### Writing Style Guidelines
- **Tone**: Friendly and informative
- **Perspective**: Use second-person ("you" and "your") for user-facing messages
- **Clarity**: Write for non-native English speakers
- **Formatting in Messages**:
  - Use backticks for: file paths, filenames, variable names, field entries
  - Use sentence case for titles and messages
  - Avoid abbreviations when possible

## Code Organization

### File Structure
```
custom_components/xenia_home/
├── __init__.py          # Entry point with async_setup_entry
├── manifest.json        # Integration metadata
├── const.py            # Domain and constants
├── config_flow.py      # UI configuration and options flow
├── coordinator.py      # Data update coordinators (fast + config)
├── entity.py          # Base entity class and shared helpers
├── xenia.py           # Device API client
├── script_parser.py   # Script instruction parser
├── sensor.py          # Sensor platform
├── binary_sensor.py   # Binary sensor platform
├── number.py          # Number platform (temperatures + weight target)
├── select.py          # Select platform
├── switch.py          # Switch platform
├── button.py          # Button platform (script execution)
├── event.py           # Event platform (shot tracking)
├── strings.json       # User-facing text and translations
└── services.yaml      # Service definitions
```

### Key Modules
- **xenia.py**: Device communication logic (HTTP API client)
- **coordinator.py**: Dual coordinator pattern — fast (1s) and config (1h)
- **entity.py**: Base entity definitions and shared `build_device_info()` helper
- **config_flow.py**: Configuration flow for UI-based setup and options flow for weight management
- **script_parser.py**: Pure Python parser for machine script instructions

### Runtime Data Storage
Use `ConfigEntry.runtime_data` to store both coordinators:
```python
type XeniaConfigEntry = ConfigEntry[XeniaRuntimeData]

async def async_setup_entry(hass: HomeAssistant, entry: XeniaConfigEntry) -> bool:
    coordinator = XeniaDataUpdateCoordinator(hass, entry, xenia)
    config_coordinator = XeniaConfigCoordinator(hass, entry, xenia)
    await coordinator.async_config_entry_first_refresh()
    await config_coordinator.async_config_entry_first_refresh()
    entry.runtime_data = XeniaRuntimeData(coordinator, config_coordinator)
```

## Async Programming

- All external I/O operations must be async
- **Best Practices**:
  - Avoid sleeping in loops
  - Avoid awaiting in loops - use `gather` instead
  - No blocking calls
  - Use `async_get_clientsession(hass)` for HTTP requests

### Data Update Coordinators
The integration uses two coordinators:
- **`XeniaDataUpdateCoordinator`** — polls every 1 second for live sensor data
- **`XeniaConfigCoordinator`** — polls every 1 hour for config data (scripts, switches, firmware, managed script)

```python
async def _async_update_data(self):
    try:
        data = await self.xenia.get_overview()
    except (ClientError, OSError, TimeoutError) as err:
        raise UpdateFailed(f"Xenia fetch failed: {err}") from err
    return data
```

## Configuration Flow

- **UI Setup Required**: Integration supports configuration via UI
- **Data Storage**:
  - Connection-critical config: Store in `ConfigEntry.data`
  - Non-critical settings: Store in `ConfigEntry.options`
- **Validation**: Always validate user input before creating entries
- **Connection Testing**: Test device connection during config flow
- **Duplicate Prevention**: Prevent duplicate configurations using unique IDs

## Entity Development

### Unique IDs
Every entity must have a unique ID for registry tracking:
```python
class XeniaSensor(CoordinatorEntity):
    def __init__(self, coordinator: XeniaDataUpdateCoordinator, device_id: str, sensor_type: str) -> None:
        super().__init__(coordinator)
        self._attr_unique_id = f"{device_id}_{sensor_type}"
```

### Entity Naming
Use `has_entity_name` pattern:
```python
class XeniaSensor(CoordinatorEntity):
    _attr_has_entity_name = True

    def __init__(self, coordinator: XeniaDataUpdateCoordinator, description) -> None:
        super().__init__(coordinator)
        self.entity_description = description
        self._attr_device_info = DeviceInfo(

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Knoedelauflauf) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
