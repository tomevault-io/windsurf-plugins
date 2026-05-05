---
trigger: always_on
description: These instructions tell GitHub Copilot Chat how to work in this repo. Assume changes target a Home Assistant custom integration that talks to Creality printers over a local WebSocket, plus a bundled Lovelace card.
---

# Copilot Instructions for this repository

These instructions tell GitHub Copilot Chat how to work in this repo. Assume changes target a Home Assistant custom integration that talks to Creality printers over a local WebSocket, plus a bundled Lovelace card.

## Project overview

- Domain: `ha_creality_ws` (custom_components/ha_creality_ws)
- Purpose: Low-latency local WebSocket telemetry and control for Creality K-series and compatible printers. Bundles a dependency-free Lovelace card.
- Connectivity: Local WebSocket (default ws://<host>:9999) with push updates; no polling.
- Discovery: Zeroconf matches for names containing creality/k1/k2.
- Python target: 3.11 (ruff target-version py311).

## Repo layout quick map

- `custom_components/ha_creality_ws/__init__.py` – HA setup, wiring coordinator and platforms; caches device info and options
- `custom_components/ha_creality_ws/coordinator.py` – DataUpdateCoordinator subtype; owns `KClient`; `wait_for_fields` helper
- `custom_components/ha_creality_ws/ws_client.py` – Resilient WebSocket client, heartbeat, jittered backoff, periodic GETs
- `custom_components/ha_creality_ws/sensor.py` – Sensors (status, temps, progress, positions, etc.)
- `custom_components/ha_creality_ws/button.py` – Pause/Resume/Stop controls
- `custom_components/ha_creality_ws/switch.py` – Light switch and similar
- `custom_components/ha_creality_ws/number.py` – Number entities (speed/flow/targets; K2 box control only)
- `custom_components/ha_creality_ws/camera.py` – MJPEG (K1) and WebRTC (K2) camera implementations
- `custom_components/ha_creality_ws/image.py` – Image platform exposing current print preview (K1 family)
- `custom_components/ha_creality_ws/light.py` – Light platform (printer chamber light)
- `custom_components/ha_creality_ws/fan.py` – Fan platform (model/case/side fans)
- `custom_components/ha_creality_ws/config_flow.py` – UI config + Options (power switch binding, camera mode, go2rtc)
- `custom_components/ha_creality_ws/entity.py` – Base entity with zeroing rules and device info
- `custom_components/ha_creality_ws/utils.py` – Helpers (numeric coercion, parsing, model detection)
- `custom_components/ha_creality_ws/services.yaml` – Custom HA services
- `custom_components/ha_creality_ws/manifest.json` – HA manifest (requirements, version, zeroconf)
- `tools/test_files/deploy_to_ha.sh` – Dev-to-HA deploy script with backup and restart

## Design anchors to preserve

- Coordinator availability model: an entity stays available and zeros when the power switch is OFF or the link is stale. Use `KEntity._should_zero()`.
- Power switch awareness: `KCoordinator.power_is_off()` drives UI zeroing and WS client start/stop.
- Pause/resume pipeline: queued actions in coordinator (`request_pause`, `request_resume`, `_flush_pending`) with non-optimistic UI.
- Status derivation: `PrintStatusSensor` maps telemetry to human-readable status. Don’t regress this mapping.
- Resilient WS client: `KClient` owns heartbeat, jittered backoff, reconnect, and periodic GETs.
- Local-first, no cloud: Never introduce cloud calls. Keep latency low and updates push-driven.
- Model-specific feature detection: conditional features by model (box temp sensor/control, light, camera type).
- Sensor zeroing when printer off: Layer sensors show 0, text sensors show "N/A", status shows "off" when power is off.
- Lovelace card behavior: chips/buttons render instantly with optimistic UI; Power chip pinned far-right and only visible when configured; Light chip visibility reacts to power state and status without reload.

## Home Assistant specifics

- Entities subclass `KEntity`; follow CoordinatorEntity pattern; no polling.
- Use `selector` in config flow options; respect existing option keys.
- For new services: declare in `services.yaml` and implement async-safe handlers in platform or `__init__.py`.
- For new simple sensors: prefer adding to `SPECS` in `sensor.py`; ensure `_should_zero()`.
- Prefer HA unit constants with compatibility fallbacks.
- Image platform: subclass `ImageEntity`; call `ImageEntity.__init__(self, hass)`; set `image_last_updated` when new bytes fetched; return placeholder bytes when content is unavailable.

## Model detection and feature management

Use `ModelDetection` which reads both `model` and `modelVersion` codes.

- Detection helpers:
  - K1 family: K1, K1C, K1 Max (not K1 SE)
  - K2 family: codes F021 (K2), F012 (K2 Pro), F008 (K2 Plus)
  - Ender 3 V3 family: F001 (V3), F002 (V3 Plus), F005 (V3 KE)
  - Creality Hi: F018
- Capabilities by model:
  - Box temperature sensor: K1 (except K1 SE), K2; not present on Creality Hi
  - Box temperature control: K2 Pro and K2 Plus only; not present on Creality Hi
  - Light: All except K1 SE and Ender 3 V3 family
  - Camera types: WebRTC (K2); MJPEG optional (K1 SE, Ender 3 V3); MJPEG default (others)
- `resolved_model()` provides a stable model name for device info caching when the friendly name is missing.

## Camera implementation

- MJPEG (K1 family):
  - Snapshot extraction from MJPEG stream; fallback tiny JPEG when unavailable
  - Live streaming via `handle_async_mjpeg_stream`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [3dg1luk43/ha_creality_ws](https://github.com/3dg1luk43/ha_creality_ws) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
