---
trigger: always_on
description: This is a Home Assistant custom integration for monitoring and controlling ByteWatt/Neovolt battery systems. The integration provides comprehensive real-time monitoring of solar, battery, and grid power flows with sophisticated recovery mechanisms and battery control capabilities.
---

# ByteWatt Home Assistant Integration Development Guide

## Repository Overview

This is a Home Assistant custom integration for monitoring and controlling ByteWatt/Neovolt battery systems. The integration provides comprehensive real-time monitoring of solar, battery, and grid power flows with sophisticated recovery mechanisms and battery control capabilities.

### Current Status
- **Version**: 1.0.0 (HACS compatible)
- **Architecture**: Production-ready with robust error handling and automatic recovery
- **Recent Cleanup**: Technical debt removed, modular architecture implemented (2024-12)

## Project Structure

```
custom_components/bytewatt/
├── __init__.py                 # Main integration entry point & services
├── bytewatt_client.py         # High-level API wrapper
├── config_flow.py             # Configuration UI flow
├── const.py                   # Constants and configuration
├── coordinator.py             # Data update coordinator with recovery
├── models.py                  # Data models for the integration
├── sensor.py                  # All sensor entity definitions
├── services.yaml              # Service definitions for Home Assistant
├── validation.py              # Minimal data validation (cleaned up)
├── manifest.json              # Integration metadata
├── translations/
│   └── en.json               # English translations
├── api/                      # Low-level API clients
│   ├── __init__.py
│   ├── neovolt_auth.py       # Authentication handling
│   ├── neovolt_client.py     # Core API client with async methods
│   └── settings.py           # Battery settings API
└── utilities/                # Utility modules (new modular architecture)
    ├── __init__.py
    ├── circuit_breaker.py    # Circuit breaker pattern implementation
    ├── connection_stats.py   # Connection health statistics
    ├── diagnostic_service.py # Health checks and diagnostics
    └── time_utils.py         # Time manipulation utilities
```

## Commands
- **Manual Install**: Copy `custom_components/bytewatt` to Home Assistant's `custom_components` directory
- **Lint**: `flake8 custom_components/bytewatt --max-line-length=100`
- **Type Check**: `mypy custom_components/bytewatt --ignore-missing-imports`
- **Validate**: `hass-config-check custom_components/bytewatt`
- **Debug**: Add to HA configuration.yaml: `logger: default: debug`
- **Syntax Check**: `python3 -m py_compile custom_components/bytewatt/**/*.py`

## Code Style
- **Python**: Version 3.9+ compatible
- **Formatting**: 4 spaces (not tabs), <100 char lines
- **Imports**: Standard lib → Third party → Home Assistant, grouped with blank lines
- **Naming**: CamelCase (classes), UPPER_CASE (constants), snake_case (variables/functions)
- **Error handling**: Try/except with appropriate logging levels
- **Comments**: Docstrings with triple double quotes
- **Type hints**: Required for all new functions/methods
- **Magic Numbers**: Use named constants from `const.py`

## Architecture

### Core Components
- **`bytewatt_client.py`**: High-level async API wrapper with battery control methods
- **`coordinator.py`**: DataUpdateCoordinator with circuit breaker pattern and automatic recovery
- **`sensor.py`**: All sensor entities (50+ sensors for comprehensive monitoring)
- **`config_flow.py`**: UI configuration flow with validation
- **`const.py`**: All constants, defaults, and configuration keys

### API Layer (`api/`)
- **`neovolt_client.py`**: Low-level async HTTP client with authentication
- **`neovolt_auth.py`**: Password encryption and authentication logic
- **`settings.py`**: Battery settings API with validation and retry logic

### Utility Modules (`utilities/`)
- **`circuit_breaker.py`**: Implements circuit breaker pattern for API resilience
- **`connection_stats.py`**: Tracks connection health and response times
- **`diagnostic_service.py`**: Health checks, diagnostics, and logging
- **`time_utils.py`**: Time format validation and manipulation

### Key Features
1. **Real-time Monitoring**: Battery SOC, power flows, energy statistics
2. **Daily Statistics**: PV generation, consumption, self-sufficiency metrics
3. **Battery Control**: Charge/discharge scheduling, minimum SOC setting
4. **Automatic Recovery**: Circuit breaker pattern with exponential backoff
5. **Health Monitoring**: Comprehensive diagnostics and connectivity checks
6. **HACS Compatible**: Installable through Home Assistant Community Store

### Services Available
- `bytewatt.set_discharge_time` - Set battery discharge end time
- `bytewatt.set_discharge_start_time` - Set battery discharge start time  
- `bytewatt.set_charge_start_time` - Set battery charge start time
- `bytewatt.set_charge_end_time` - Set battery charge end time
- `bytewatt.set_minimum_soc` - Set minimum state of charge
- `bytewatt.update_battery_settings` - Update multiple settings at once
- `bytewatt.force_reconnect` - Force API reconnection
- `bytewatt.health_check` - Run comprehensive health check
- `bytewatt.toggle_diagnostics` - Enable/disable diagnostic logging

### Configuration Options

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [candreacchio/neovoltBattery_HomeAssistantPlugin](https://github.com/candreacchio/neovoltBattery_HomeAssistantPlugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
