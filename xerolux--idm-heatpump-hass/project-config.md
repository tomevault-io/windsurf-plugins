---
trigger: always_on
description: This file provides guidance for AI assistants working on this codebase.
---

# AGENTS.md — IDM Heatpump for Home Assistant

This file provides guidance for AI assistants working on this codebase.

## Project Overview

**IDM Heatpump** is a Home Assistant custom integration for controlling and monitoring IDM Navigator 1.0 / 1.7 / 2.0 / 10 / Pro heat pumps via Modbus TCP and an optional local web supplement (the 1.x controllers are served over Modbus coils, the 2.0/10/Pro families over the register map plus the web interface). It is an unofficial community project providing 100% local control (no cloud dependency).

- **Domain**: `idm_heatpump`
- **Current Version**: `0.20.1` (source-tree manifest; [latest stable release](https://github.com/Xerolux/idm-heatpump-hass/releases/latest))
- **Quality Scale**: Gold (targets official Home Assistant Core integration standards)
- **License**: MIT
- **Min HA Version**: 2026.8.1
- **Python**: 3.14+ (Home Assistant 2026.8 requires `>=3.14.2`)
- **Direct Modbus Runtime**: `modbus-connection>=4.12.3`, `tmodbus[async-serial]>=0.6.2`
- **Device Logic**: `idm-heatpump-api[web]==2.14.0` (owns its own exception hierarchy; pymodbus is no longer a dependency)
- **Open improvement plan**: `docs/dev/code-audit-2026-09.md` — the reviewed list of defects and
  cleanups with a work package per fix. Read it before starting unrelated refactoring; pick a
  package from it instead of inventing one. The 0.20.0 web-first line is documented in
  `docs/dev/ws-first-roadmap.md`.

---

## Repository Structure

```
/
├── custom_components/idm_heatpump/   # Main integration package
│   ├── __init__.py                   # Domain setup, platform loading, entry lifecycle
│   ├── manifest.json                 # Integration metadata & HA version requirements
│   ├── const.py                      # Constants, enums, option keys, defaults
│   ├── config_flow.py                # UI config flow (user, options, zones, reconfigure, web-only fallback)
│   ├── connection_entities.py         # Diagnostic entities: effective connection mode + web liveness
│   ├── coordinator.py                # DataUpdateCoordinator (polling, web supplement, writes)
│   ├── entity.py                     # Base entity class (IdmEntity)
│   ├── device_hierarchy.py           # Opt-in sub-device scopes, placement and registry wiring for the entity hierarchy
│   ├── sensor.py                     # Sensor platform (Modbus + web-only sensors + technician codes)
│   ├── binary_sensor.py              # Binary sensor platform
│   ├── number.py                     # Number platform (setpoints, GLT values)
│   ├── select.py                     # Select platform (mode registers)
│   ├── switch.py                     # Switch platform (boolean writable registers)
│   ├── climate.py                    # Climate platform (heating circuits, zone-module rooms)
│   ├── water_heater.py               # Water heater platform (DHW setpoint)
│   ├── button.py                     # Button platform (acknowledge errors)
│   ├── services.py                   # Custom HA services (set_system_mode, acknowledge_errors, write_register)
│   ├── services.yaml                 # Service schema definitions
│   ├── diagnostics.py                # HA diagnostics export
│   ├── repairs.py                    # Repair flows (e.g. missing web PIN)
│   ├── registers.py                  # Collects entity descriptions from idm-heatpump-api
│   ├── library_adapter.py            # Adapter between idm-heatpump-api and HA EntityDescriptions
│   ├── model_resolution.py           # Pure rules deciding the Navigator model and register map from probe/stored/web/override
│   ├── modbus_client.py              # API client adapter routing raw I/O through the local transport
│   ├── modbus_transport.py           # Backend-neutral contract + modbus-connection/tmodbus implementation
│   ├── versions.py                   # Runtime dependency versions for logs, sensors, and diagnostics
│   ├── ha_compat.py                  # Version-tolerant imports across the supported HA range (vol/probatio, moved enums)
│   ├── recovery_watchdog.py          # Self-healing reload for entries in setup retry once the endpoint answers again
│   ├── adapter_descriptions.py       # HA description helpers (icons, device classes)
│   ├── adapter_enums.py              # Enum slug maps and translation keys
│   ├── entity_names.py               # Entity translation keys, placeholders and canonical English names
│   ├── adapter_names.py              # German entity names (source of truth for the de translations)
│   ├── adapter_registers.py          # Register-map filtering by model
│   ├── adapter_glt.py                # GLT measurement detection helpers
│   ├── web_data.py                   # Optional local Navigator web supplement client
│   ├── web_demand_reason.py          # Demand-reason extraction from the Nav 10 WebSocket home/detail frame
│   ├── web_demand_reason_entities.py # Entities publishing the controller demand reason / PV flag
│   ├── room_temp_forwarding.py       # Forward HA room temperatures (per circuit) and humidity (global) to GLT registers
│   ├── external_power_forwarding.py  # Optional PV, consumption and battery forwarding to IDM GLT registers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Xerolux/idm-heatpump-hass](https://github.com/Xerolux/idm-heatpump-hass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
