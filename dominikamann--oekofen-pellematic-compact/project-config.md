---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A Home Assistant **custom component** (`domain: oekofen_pellematic_compact`) that talks locally to Ökofen Pellematic Compact heaters via their TCP/JSON interface (`http://<ip>:<port>/<password>/all`). Distributed via HACS. The component lives entirely in `custom_components/oekofen_pellematic_compact/`.

## Commands

```bash
# Activate the local dev environment (Home Assistant is pre-installed here)
source .venv/bin/activate

# Run Home Assistant against the test config (uses ./config and serves on :8123)
./start_homeassistant.sh
# or:   hass -c config --debug

# Tests
pytest tests/                                  # all tests
pytest tests/test_discovery.py -v              # one file
pytest tests/test_discovery.py::test_name -v   # one test
pytest tests/unit/                             # fast unit tests only

# Tail integration-relevant log lines
./monitor_logs.sh
```

There is no lint/format config — don't add one without checking. Hassfest validation runs in CI (`.github/workflows/hassfest Action.yml`).

## Architecture

### Dynamic discovery is the core idea

All sensor/select/number/binary_sensor definitions are **discovered at runtime from API metadata** — there are no hard-coded entity lists. The old approach (~1300 lines of `*_SENSOR_TYPES` constants) was removed; see the note at the bottom of `const.py`. When adding support for new Ökofen features, do **not** add per-key definitions — extend the discovery logic instead.

The pipeline:

1. **`__init__.py`** — `PellematicHub.fetch_pellematic_data()` polls the API on `scan_interval`. The Ökofen API requires ≥ 2500 ms between requests; the hub enforces a 2.5 s minimum interval per-instance (rate limiter in `fetch_pellematic_data`).
2. **`dynamic_discovery.py`** — `discover_all_entities(api_data)` walks the API response and classifies each field:
   - Keys starting with `L_` → read-only sensor (or binary sensor if `format` is `0:x|1:y`)
   - Keys without `L_` that match `is_read_only_statistic(...)` (totals, runtimes, `_yesterday`, etc.) → sensor, not number
   - Writable + `format` with >2 options → select
   - Writable + `format` with 2 options → select (not binary, because it's settable)
   - Writable + `min`/`max` → number
   - Otherwise → sensor fallback
3. **Platform files** (`sensor.py`, `binary_sensor.py`, `select.py`, `number.py`, `climate.py`) — each `async_setup_entry` calls `discover_all_entities()` via the shared `setup_platform_with_retry()` helper in `__init__.py`. If the API hasn't returned data yet, setup is retried every 60 s; this is why entities can appear up to a minute after install.
4. **Binary sensors are their own platform** (`binary_sensor.py`, `PLATFORMS = ["sensor", "binary_sensor", "select", "number", "climate"]`). `PellematicBinarySensor` itself is still defined in `sensor.py` (historic location, imported by `binary_sensor.py`). Older versions registered them on the `sensor` platform, so existing installs have orphan `sensor.*` entries — these are surfaced via the Repairs platform (`repairs.py`); the fix flow deletes the orphans and reloads the entry so `binary_sensor.py` recreates them. Auto-renaming via `entity_registry.async_update_entity` is **not** an option — HA forbids cross-domain renames (`raise ValueError("New entity ID should be same domain")` in `entity_registry.py`).

### API quirks the code must handle

These are workarounds for real firmware behavior — when touching the request/parse path, keep them:

- **Two response shapes:** modern firmware returns `{"val": 123, "unit": "°C", "factor": 0.1, ...}`; old firmware (≤ v3.10d) returns bare values. Detected by `_api_response_has_metadata()` in `__init__.py`. Use `get_api_value()` from `const.py` to read either shape.
- **API suffix:** `?` vs `??` is **decoupled from metadata** (issue #191). Most firmware uses `?`; some old Euro firmware exposes richer metadata only via `??`; some US 3.10 firmware *drops the connection* on `??` and must use `?` even though it returns bare values. `_detect_api_config()` chooses empirically — probe `?`, only switch to `??` if it yields strictly richer (metadata) data or `?` failed — preferring `?`. The `old_firmware` flag now only means "response has no metadata" and never forces the suffix. Stored as `CONF_API_SUFFIX`. Mirror any change in `config_flow.py::_fetch_api_data` (auto-detect branch).
- **Charset:** can be UTF-8 or ISO-8859-1, sometimes mixed within one response. `_detect_api_config` uses a 20%-replacement-character heuristic. The default is `iso-8859-1` for safety.
- **Invalid JSON in responses:**
  - `L_statetext:` → `L_statetext":` (firmware 4.02 bug)
  - Unescaped control characters inside string values (`\n`, `\r`, `\t`) — escaped via regex in `fetch_data()` and in `tests/conftest.py::load_fixture` (keep them in sync).
- **Sentinel values:** `_sanitize_oekofen_value()` in `sensor.py` drops near-min/near-max values when range > 1000 (e.g. `32767`, `-32768`).

### Component prefix convention

Auto-discovered components use these prefixes (`discover_components_from_api` in `__init__.py`):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dominikamann/oekofen-pellematic-compact](https://github.com/dominikamann/oekofen-pellematic-compact) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
