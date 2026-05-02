---
trigger: always_on
description: This repository contains a Home Assistant custom integration for **Marstek energy storage devices** (Venus A/D/E 3.0, etc.) using the **local “Open API” over UDP**.
---

# Agent Protocol: Marstek (Home Assistant Custom Integration)

This repository contains a Home Assistant custom integration for **Marstek energy storage devices** (Venus A/D/E 3.0, etc.) using the **local “Open API” over UDP**.

## What this integration is (and is not)

- **Domain**: `marstek`
- **Scope**: Marstek devices that support **OPEN API** on the local network.
- **Transport**: UDP JSON-RPC-like messages (default port **30000**) as described in `docs/marstek_device_openapi.MD`.
- **Pattern**: `local_polling` using a single `DataUpdateCoordinator` per device + a background `Scanner` to detect IP changes.
- Quality Scale: Silver (aiming for Gold with >95% coverage; tracked in `quality_scale.yaml`)

Important compatibility notes:
- The integration is currently **not compatible with Venus E2.0** devices (see `README.md`).

## Research before implementation

For any question or request, always run the `runSubagent` Official Docs Researcher (as defined in `.github/agents/official-docs-researcher.agent.md`) to do online research before starting the implementation. Summarize the findings using official sources only, include a direct link for every key claim, and keep those links or citations in the task notes. If the vendor, version, or goal is ambiguous, ask clarifying questions before proceeding. 

Note: When asked about Marstek information, that exists in our local API documentation in `docs/marstek_device_openapi.MD` as there are no online resources for that.

Note: If you have already done research via the Official Docs Researcher, you do not have to run it again.
Note: For release management you do not have to do Online Resarch, the "Release Management" skill is enough

## Architectural constraints you must respect

### 1) Keep polling centralized
- Do **not** add per-entity polling or extra network calls from entities.
- All entities must read from `MarstekDataUpdateCoordinator.data`.
- Avoid concurrent requests; Marstek devices can be sensitive to request bursts.
- Coordinator uses **tiered polling** (fast/medium/slow intervals) to reduce device load.

If you add/modify device control:
- Pause coordinator polling while sending control commands (see `custom_components/marstek/device_action.py`) to avoid concurrent UDP traffic.

### 2) Async-only I/O
- Never use blocking I/O.
- All network operations must be awaited and run in the event loop.

### 3) Home Assistant integration contract
- Config/UI-first: no new YAML setup; keep config/reauth/options in `config_flow.py` with selectors where it improves UX.
- Unique IDs must stay stable (BLE-MAC-based) to prevent duplicate entities on IP changes.
- Add user-facing text via `strings.json` → mirrored to `translations/en.json`; prefer descriptive error keys (e.g., `cannot_connect`).
- Entity classes must set `_attr_has_entity_name = True` and expose `device_info` for proper device grouping.
- Use `EntityDescription` dataclasses for declarative sensor/binary_sensor definitions.
- Loading must be async and non-blocking; any sync library work belongs in executor jobs.
- Use `entry.async_on_unload()` for cleanup callbacks.
- Services must be registered idempotently (check `hass.services.has_service()` first).

### 4) Discovery and IP changes are handled by the scanner
- Setup/connection uses the configured IP; it does **not** perform discovery during setup.
- `MarstekScanner` runs periodic broadcast discovery and triggers an integration discovery flow to update the config entry when IP changes.
- Don’t add “fallback discovery” inside coordinator updates; it creates race conditions and extra traffic.

### 5) OPEN API semantics (UDP)
- Devices must have OPEN API enabled in the Marstek app.
- Default UDP port is 30000; the Open API spec recommends using a high port range.
- LAN discovery uses UDP broadcast + `Marstek.GetDevice` (see `docs/marstek_device_openapi.MD`).
- Shared UDP client is stored in `hass.data[DOMAIN][DATA_UDP_CLIENT]` and reused across entries.

## Code map (where to implement changes)

| Concern | File | Notes |
|---|---|---|
| Setup / teardown | `__init__.py` | Creates shared UDP client + coordinator; starts `MarstekScanner`; forwards platforms; uses `entry.async_on_unload()` |
| Config flow | `config_flow.py` | Broadcast discovery UI, DHCP updates, reauth, reconfigure, options flow with sections |
| Polling + error handling | `coordinator.py` | Single source of truth; tiered polling (fast/medium/slow); returns previous data on connectivity issues |
| IP change detection | `scanner.py` | Periodic broadcast discovery (60s); triggers discovery flow to update config entries |
| Sensors | `sensor.py` | EntityDescription pattern; coordinator-backed; stable unique IDs; `suggested_display_precision` |
| Binary sensors | `binary_sensor.py` | EntityDescription pattern; CT connection status |
| Select entities | `select.py` | Operating mode selection (Auto/AI/Manual/Passive) |
| Services | `services.py` | Idempotent registration; passive mode, manual schedules, data sync |
| Device actions | `device_action.py` | Automation actions using `ES.SetMode` with retries + verification; pauses polling |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [taurgis/has-marstek-local-api](https://github.com/taurgis/has-marstek-local-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
