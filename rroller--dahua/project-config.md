---
trigger: always_on
description: <!-- tags: ai-context, codebase-navigation, dahua, home-assistant, custom-integration -->
---

# AGENTS.md — Dahua Home Assistant Integration

<!-- tags: ai-context, codebase-navigation, dahua, home-assistant, custom-integration -->

Custom Home Assistant integration for Dahua IP cameras, doorbells (VTO), NVRs, and DVRs. Also supports Amcrest, Lorex, IMOU, EmpireTech, and Avaloid Goliath rebranded devices. Version 0.9.81, HACS-installable.

## Table of Contents

- [Directory Map](#directory-map) — Where to find things
- [Architecture Overview](#architecture-overview) — How the system fits together
- [Key Entry Points](#key-entry-points) — Where to start reading
- [Device-Specific Branching](#device-specific-branching) — Model detection patterns
- [Event System](#event-system) — Dual event streaming architecture
- [API Layer](#api-layer) — Three distinct API clients
- [Config and CI](#config-and-ci) — Tooling discoverable from config files
- [Known Quirks](#known-quirks) — Non-obvious behaviors
- [Detailed Documentation](#detailed-documentation) — Deep-dive reference files
- [Custom Instructions](#custom-instructions) — Human/agent-maintained conventions

## Directory Map

<!-- tags: navigation, directory-structure -->

```
custom_components/dahua/
├── __init__.py          # Coordinator (central hub): setup, polling, event streams, feature detection
├── client.py            # HTTP API client: all CGI endpoint calls, Digest Auth, RTSP URLs
├── camera.py            # Camera entity + 18 service registrations (infrared, overlays, IVS, PTZ, reboot)
├── binary_sensor.py     # Event-driven binary sensors (motion, tripwire, doorbell press)
├── switch.py            # Toggles: motion detection, siren, smart motion, disarming
├── light.py             # IR light, illuminator, flood light, security light, ring light
├── select.py            # Dropdowns: doorbell light mode, PTZ preset position
├── config_flow.py       # UI setup flow: credentials → name → config entry
├── entity.py            # Base entity: device info, unique ID from serial number
├── vto.py               # VTO doorbell binary TCP client (port 5000, DHIP protocol)
├── rpc2.py              # Alternative RPC2 JSON API client (currently unused by main flow)
├── digest.py            # Custom aiohttp Digest Auth (aiohttp lacks native support)
├── dahua_utils.py       # Brightness conversion, event stream text parser
├── const.py             # Domain, platform list, config keys, icons
├── models.py            # CoaxialControlIOStatus dataclass (used by rpc2.py only)
├── services.yaml        # HA service definitions for all 18 camera services
└── translations/        # UI strings: en, es, fr, it, bg, ca, nl, pt, pt-BR
```

Other directories: `tests/` (minimal), `scripts/` (develop, lint, setup), `.github/workflows/` (CI).

## Architecture Overview

<!-- tags: architecture, design-patterns -->

```mermaid
graph LR
    CF[Config Flow] -->|creates| COORD[Coordinator<br/>__init__.py]
    COORD -->|polls via| CLIENT[DahuaClient<br/>client.py]
    COORD -->|streams via| CLIENT
    COORD -->|VTO events via| VTO[VTOClient<br/>vto.py]
    COORD -->|manages| ENT[Entities<br/>camera, sensor, switch, light, select]
    CLIENT -->|HTTP+DigestAuth| DEV[Camera CGI API]
    VTO -->|TCP:5000 binary| DEV2[Doorbell DHIP]
```

**Coordinator pattern**: `DahuaDataUpdateCoordinator` (extends HA's `DataUpdateCoordinator`) is the single source of truth. It initializes by probing device capabilities, polls state every 30s, and maintains persistent event stream connections.

**Entity hierarchy**: All entities extend `DahuaBaseEntity` → `CoordinatorEntity`. Device info (model, serial, firmware) comes from the coordinator.

## Key Entry Points

<!-- tags: entry-points, getting-started -->

| Task | Start Here |
|------|-----------|
| Understand the integration | `__init__.py` → `DahuaDataUpdateCoordinator` |
| Add a new API call | `client.py` → `DahuaClient` |
| Add a new entity | Create in appropriate platform file, register in `__init__.py` PLATFORMS |
| Add a new service | `camera.py` → `async_setup_entry()` service registrations + `services.yaml` |
| Add a new event type | `config_flow.py` → `ALL_EVENTS` list |
| Modify VTO/doorbell behavior | `vto.py` → `DahuaVTOClient` |
| Change feature detection | `__init__.py` → `_async_update_data()` initialization block |

## Device-Specific Branching

<!-- tags: device-detection, model-checks -->

The coordinator uses model string matching to determine device capabilities. Key methods:

| Method | Detection Logic |
|--------|----------------|
| `is_doorbell()` | Model starts with VTO, DH-VTO, DHI, AD, DB6, DB2X, AV-V |
| `is_amcrest_doorbell()` | Model starts with AD or DB6 |
| `supports_siren()` | Model contains -AS-PV, L46N, or starts with W452ASD |
| `supports_security_light()` | Model contains -AS-PV, or is AD410, DB61i, IP8M-2796E |
| `is_flood_light()` | Model starts with ASH26, V261LC, W452ASD, or contains L26N, L46N |
| `supports_infrared_light()` | Lighting supported AND model doesn't contain -AS-PV, -AS-NI, LED-S2 |

Adding support for a new device model typically means updating these string checks.

## Event System

<!-- tags: events, event-streaming -->

Two parallel event streaming mechanisms:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rroller/dahua](https://github.com/rroller/dahua) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
