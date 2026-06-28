---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Home Assistant **custom integration** (HACS-distributed) that monitors and controls a TrueNAS device. All integration code lives under [custom_components/truenas/](custom_components/truenas/). It targets TrueNAS 25.04+ and Home Assistant 2024.8.0+, and communicates with TrueNAS exclusively over its JSON-RPC **WebSocket** API (`local_polling`, 60s interval).

## Commands

Linting/formatting and tooling mirror CI ([.github/workflows/ci.yml](.github/workflows/ci.yml)):

```bash
ruff check .            # lint (rules E, F, W, I, UP, ASYNC; py313, line-length 88)
ruff format --check .   # formatting check; drop --check to auto-format
bandit -r custom_components/truenas   # security scan (part of CI)
```

- Python target is **3.13**.
- `pytest` is configured in [pyproject.toml](pyproject.toml) (`asyncio_mode=auto`, `testpaths=["tests"]`), but there is currently **no `tests/` directory** and the pytest step in CI is commented out. If adding tests, create `tests/` and re-enable that CI step.
- CI also runs Home Assistant **hassfest** validation and HACS validation on push/PR.
- Runtime deps come from [manifest.json](custom_components/truenas/manifest.json) (`websockets>=15.0.1`); dev deps are in the [Pipfile](Pipfile). `.github/generate_requirements.py` regenerates `requirements*.txt` from the Pipfile during CI.
- Bumping the release: `version` in [manifest.json](custom_components/truenas/manifest.json) is the source of truth (`.github/update_version.py` updates it during release).

## Architecture

Data flows in one direction: **API → Coordinator → `coordinator.ds` dict → entities**.

### Layers

- **[api.py](custom_components/truenas/api.py) — `TrueNASAPI`**: Synchronous (thread-based, not asyncio) WebSocket client. `query(method, params)` issues a JSON-RPC call and returns the unwrapped `result` (or `None` on any error). It handles `connect`/login (`auth.login_with_api_key`), subprotocol negotiation quirks, and a large catalogue of connection-error classification (see `ERR_*` constants in [const.py](custom_components/truenas/const.py)). Because it is synchronous, the coordinator always calls it via `hass.async_add_executor_job(...)`. **Concurrency:** two reentrant locks — `_io_lock` (all websocket recv/send) must always be acquired **before** `_lock` (fast state writes). All websocket I/O goes through `_recv_locked`/`_send_locked`; preserve this discipline when editing.

- **[coordinator.py](custom_components/truenas/coordinator.py) — `TrueNASCoordinator`**: A `DataUpdateCoordinator`. `_async_update_data` runs a list of `get_*` jobs concurrently via `asyncio.gather` (each wrapped to swallow/log exceptions), then runs `get_pool` last because it depends on dataset data. Every `get_*` method calls one or more `self.api.query(...)` and stores normalized results into `self.ds[<domain>]` (e.g. `system_info`, `disk`, `pool`, `dataset`, `vm`, `container`, `app`, `service`, `directoryservices`, `cloudsync`, `replication`, `rsynctask`, `snapshottask`, `cronjob`, `alerts`, `ups`, `interface`). The update check (`get_updatecheck`) runs at most every 12 hours. The whole `self.ds` dict is the coordinator's `data`.

- **[apiparser.py](custom_components/truenas/apiparser.py) — `parse_api`**: The central normalization helper used by nearly every `get_*` method. It maps raw API entries into a flat dict keyed by `key` (e.g. `id`, `guid`, `identifier`). Field specs (`vals`) support nested source paths via `/` (e.g. `"scan/start_time/$date"`), type coercion (`"bool"`), defaults, and `convert: "utc_from_timestamp"`. `ensure_vals` guarantees keys exist with defaults even when absent from the source. When adding a new monitored field, add a spec entry here rather than post-processing.

- **[entity.py](custom_components/truenas/entity.py) — `TrueNASEntity` + `async_add_entities`**: Shared base entity (a `CoordinatorEntity`) and the generic platform setup. `async_add_entities` is reused by all platforms: it reads each platform module's `SENSOR_TYPES`/`SENSOR_SERVICES`, uses a per-platform **`dispatcher`** dict (string class-name → class) to instantiate entities, dynamically creates entities for each `uid` under `data_path`, and registers entity services. New entities appear on the fly via the `"update_sensors"` dispatcher signal. Device grouping and unique IDs are derived from the entity description's `ha_group`/`ha_connection`/`data_reference` fields.

### Platforms and the entity-description pattern

Platforms registered in `PLATFORMS` ([const.py](custom_components/truenas/const.py)): `SENSOR`, `BINARY_SENSOR`, `BUTTON`, `UPDATE`, `SWITCH`. Each platform has two files:

- `<platform>.py` — the entity classes and `async_setup_entry` (which only builds the `dispatcher` and calls the shared `async_add_entities`/`setup_entities`).
- `<platform>_types.py` — exports `SENSOR_TYPES` (a tuple of frozen-dataclass entity descriptions subclassing the HA `*EntityDescription`) and `SENSOR_SERVICES`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kayl-codes/homeassistant-truenas](https://github.com/kayl-codes/homeassistant-truenas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-28 -->
