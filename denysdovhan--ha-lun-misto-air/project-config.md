---
trigger: always_on
description: Agents act as senior Python collaborators. Keep responses concise,
---

# AI Coding Agents Guide

## Purpose

Agents act as senior Python collaborators. Keep responses concise,
clarify uncertainty before coding, and align suggestions with the rules linked below.

## Project Overview

This repository is a Home Assistant custom integration providing air quality data from [LUN Misto Air](https://lun.ua/misto/air). Main codebase lives under `custom_components/lun_misto_air`.

### Code structure

- `translations` — JSON translations per locale.
- `__init__.py` — sets up the integration, creates per-station coordinators and stores them in `entry.runtime_data.coordinators`, runs migrations v1→v2→v3, forwards sensor platform.
- `api.py` — HTTP client and `LUNMistoAirStation` dataclass, no HA dependencies.
- `config_flow.py` — config/options flows, creates station subentries (static/dynamic).
- `const.py` — constants and defaults; check here before adding new strings.
- `coordinator.py` — `LUNMistoAirCoordinator` fetches data for a station (static by name or nearest dynamic) and surfaces a `LUNMistoAirStation` in `data`.
- `entity.py` — base entity descriptors/device info for sensors; ensures unique IDs with `{subentry_id}-{sensor_key}`.
- `migrations.py` — data migrations; keep in sync with `CONFIG_ENTRY_VERSION` in `config_flow.py`.
- `manifest.json` — HA manifest.
- `sensor.py` — sensor entities bound to coordinators via `config_subentry_id`.
- `data.py` — runtime container (`LUNMistoAirRuntimeData`) and typed entry alias.
- `diagnostics.py` — exposes `async_get_config_entry_diagnostics` with entry metadata, subentries, coordinators, and station snapshots for HA diagnostics download.

### Using Coordinator to Fetch Data

We use one DataUpdateCoordinator per station subentry (static or dynamic) to fetch data from the LUN Misto Air API. Coordinators are created in `__init__.py` during setup and stored in `entry.runtime_data.coordinators` keyed by `subentry_id`. Platforms (e.g., `sensor.py`) read from this mapping and pass `config_subentry_id=subentry_id` when adding entities so entities bind to the right coordinator.

The coordinator:

- Receives the shared API client and subentry metadata (dependency injection).
- Fetches measurements for the configured station; picks dynamic vs static source.
- Computes derived values (AQI, last_updated, attribution) so platforms read ready values.
- Prefer logging and returning `None`/fallbacks instead of raising; add debug logs when behavior is unclear.
- Keep coordinator instances in `entry.runtime_data.coordinators`; avoid globals/singletons.

Documentation: https://developers.home-assistant.io/docs/integration_fetching_data

### Decouple API Data from Coordinator

Coordinator should not rely on API response structure. Instead, transform data into plain Python objects (e.g., dataclasses) on API class level, so coordinator only calls API methods and works with stable data structures.

### API

External API: `https://misto.lun.ua/api/v1/air/stations` (cloud polling). No auth. All HTTP and parsing lives in `api.py`.

## Runtime Data

- `LUNMistoAirRuntimeData` is a small container attached to each `ConfigEntry` under `entry.runtime_data`.
  - Fields:
    - `api`: a single shared `LUNMistoAirApi` client per entry.
    - `coordinators`: a dict keyed by `subentry_id` holding one `LUNMistoAirCoordinator` per station subentry.
- `type LUNMistoAirConfigEntry = ConfigEntry[LUNMistoAirRuntimeData]` is a typed alias used in function signatures for clarity.
- Initialization happens in `__init__.py`:
  - Build the API client, set `entry.runtime_data = LUNMistoAirRuntimeData(api=api)`.
  - For each station subentry, create a coordinator, refresh it, then store under `entry.runtime_data.coordinators[subentry_id]`.
- Platforms (e.g., `sensor.py`) retrieve coordinators via `config_entry.runtime_data.coordinators` and add entities with the appropriate `config_subentry_id`.

This pattern keeps setup logic centralized, avoids globals, and makes platform code simple and testable.

## Workflow

- Python deps defined in `pyproject.toml`, locked in `uv.lock`; manage env with `uv`.
- CI (`lint.yml`, `validate.yml`) installs uv via `astral-sh/setup-uv` and runs tools with `uv run`.
- Use `scripts/bootstrap` for fresh setup (installs uv via pipx if missing, syncs deps, installs pre-commit).
- Prefer running tooling via `uv run <tool>` to match the locked environment.

<instruction>Keep this guide synced when tooling, workflows, or runtime data structures change.</instruction>

This project is developed from Devcontainer described in `.devcontainer.json` file.

- **Adding/changing data fetching**
  - Extend `api.py` first; return Python objects (dataclasses) independent of raw JSON.
  - Use/extend `coordinator.py` to pick dynamic vs static station and compute derived values.
  - Keep all shared runtime stuff (API client, coordinators map) in `data.py` via `entry.runtime_data`.
- **Entities and platforms**
  - Add new sensor descriptors in `sensor.py` (use `translation_key`).
  - Unique ID is `{subentry_id}-{sensor_key}`; do not hardcode unique IDs in config flow.
  - Device naming relies on `CONF_NAME` (guaranteed by migrations); placeholders are provided via `device_info`.
- **Config flow**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [denysdovhan/ha-lun-misto-air](https://github.com/denysdovhan/ha-lun-misto-air) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
