---
trigger: always_on
description: A short orientation for AI coding agents (Claude, Codex, Copilot, etc.)
---

# AGENTS.md

A short orientation for AI coding agents (Claude, Codex, Copilot, etc.)
working in this repo. For human contributor guidelines see
[CONTRIBUTING.md](CONTRIBUTING.md).

## What this repo is

A Home Assistant custom integration that surfaces gas-station prices
(and optionally EV-charger details) for individual GasBuddy stations.
It depends on [py-gasbuddy](https://github.com/firstof9/py-gasbuddy) for
the upstream GraphQL client.

```
custom_components/gasbuddy/
├── __init__.py        # async_setup_entry / async_unload_entry / migrate
├── coordinator.py     # DataUpdateCoordinator wrapping py_gasbuddy
├── config_flow.py     # ConfigFlow + OptionsFlow + reconfigure
├── sensor.py          # SensorEntity definitions
├── services.py        # 5 services: lookup_gps/zip, ev_lookup_*, clear_cache
├── const.py           # CONF_* keys + SENSOR_TYPES catalog
├── diagnostics.py     # Config-entry + device diagnostics
├── strings.json       # English source + translation schema
└── translations/      # en.json (mirror of strings), fr.json, pt.json
```

## Environment + toolchain

- **Python**: target is 3.14 (`target-version = "py314"` in `pyproject.toml`).
  CI runs against Python 3.14.2.
- **Linting**: ruff via pre-commit, pinned to **v0.15.14** (see `.pre-commit-config.yaml`).
  Install that exact version locally. `pyproject.toml` ignores `PLW0717`,
  a rule selector that only exists in ruff 0.15.14+; an older ruff (e.g.
  0.15.12) fails to even parse the config with `Unknown rule selector:
  PLW0717`.
- **Formatting**: ruff format with `target-version = "py314"`. This means
  PEP 758 parenthesis-free `except`: ruff will rewrite
  `except (TypeError, ValueError):` to `except TypeError, ValueError:`.
  That's valid Python 3.14 — don't fight it.
- **Tests**: `pytest` + `pytest-homeassistant-custom-component`. The
  full suite is ~99 tests, runs in 15–25 seconds.

```bash
uv venv --python 3.14
uv pip install -r requirements_test.txt
uv pip install 'ruff==0.15.14'
.venv/bin/python -m pytest -q --no-header
.venv/bin/ruff check custom_components/
.venv/bin/ruff format --check custom_components/
```

## Architectural notes that won't be obvious from the code

### CSRF token cache is shared

`py_gasbuddy.GasBuddy` accepts a `cache_file` path. The coordinator,
config-flow, and services must all use the **same** path — defined
once as `CACHE_FILE_NAME` in `const.py`. Each module has a small
`_cache_path(hass)` helper that returns `f"{hass.config.config_dir}/{CACHE_FILE_NAME}"`.

If you construct a `GasBuddy` without passing `cache_file=`, py-gasbuddy
falls back to `~/.cache/py_gasbuddy/token` and you'll re-fetch a CSRF
token on every call — which is exactly what Cloudflare blocks on first
install. Don't do that.

### Config-flow → entry shape

Three flows create a config entry: `manual`, `home2` (search by home
coordinates), and `station_list` (search by postal). All three set
`unique_id = str(station_id)` and call `_abort_if_unique_id_configured()`
before `async_create_entry` to prevent duplicates.

Stored in `entry.data`: `CONF_STATION_ID`, `CONF_NAME`, `CONF_SOLVER`,
`CONF_TIMEOUT`, `latitude`, `longitude`.

Stored in `entry.options`: `CONF_INTERVAL`, `CONF_UOM`, `CONF_GPS`,
`CONF_EV_CHARGING`, `CONF_FETCH_GAS`. `CONFIG_VER` is bumped when this
shape changes; see `async_migrate_entry`.

### Virtual Hub Configuration

A special config entry with `unique_id = "hub"` acts as the Virtual Hub.
- **Global Settings**: It holds global settings (`solver`, `timeout`, and `brand_adjustments`).
- **Coordinator Fallback**: The update coordinator retrieves these settings via `_get_hub_setting(key, default)` dynamically. If no Hub is configured, it falls back to the individual station's configuration.
- **UI & Device Nesting**: The hub registers a device in the Home Assistant device registry. Station devices link to the Hub using `via_device=(DOMAIN, "hub")` in their `DeviceInfo` to nest them under the Hub device in the UI.
- **Automatic Migration**: A setup routine automatically imports settings from existing station entries into the Hub upon creation, and cleans them up from the station entries.

### Coordinator's two-path update

`_async_update_data` honors `CONF_FETCH_GAS`:

- **If `fetch_gas=False`** (typically an EV-only station): bootstrap
  `self._data` with the station id + config coordinates, then fall
  through to the EV enrichment block. Don't call `price_lookup` — it
  would `APIError` every poll for EV-only stations.
- **If `fetch_gas=True`**: call `price_lookup`. On failure, if
  `ev_charging` is also enabled, fall back to `ev_stations_nearby` to
  resolve the station as an EV-only entry.

When the EV-fallback path constructs synthetic data, **carry forward
`unit_of_measure` and `currency` from the previous `self._data` rather
than hardcoding USD/gallon** (Canadian stations report CAD/cents-per-liter).

### Cloudflare-block detection

When py-gasbuddy can't fetch a CSRF token (typically because Cloudflare
is blocking and no FlareSolverr is configured), the GraphQL request
fails with a content-less `LibraryError`. We can't tell that apart from
"GasBuddy returned an error for this station ID" by inspecting the
exception alone.

Current workaround in `validate_station`: probe the client's

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [firstof9/ha-gasbuddy](https://github.com/firstof9/ha-gasbuddy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
