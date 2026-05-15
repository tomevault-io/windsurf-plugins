---
trigger: always_on
description: Home Assistant custom integration that schedules appliances during cheapest (or most expensive) electricity hours using Nord Pool prices. Supports two scheduling strategies: **Lowest Price** (fixed-period optimization) and **Minimum Runtime** (rolling-window scheduling).
---

# CLAUDE.md — Power Saver Integration

Home Assistant custom integration that schedules appliances during cheapest (or most expensive) electricity hours using Nord Pool prices. Supports two scheduling strategies: **Lowest Price** (fixed-period optimization) and **Minimum Runtime** (rolling-window scheduling).

## Post-Change Checklist

- **Always review [README.md](README.md) after any changes** to see if it needs updating to reflect the changes made (new features, config options, sensors, behavior changes, etc.)
- Update [strings.json](custom_components/power_saver/strings.json) AND [translations/en.json](custom_components/power_saver/translations/en.json) when adding/changing UI text or entities
- Update [translations/sv.json](custom_components/power_saver/translations/sv.json) for Swedish translations

## Commands

```bash
# Install test dependencies
pip install -r dev/requirements_test.txt

# Run all tests
pytest tests/

# Run specific test file
pytest tests/test_scheduler.py

# Run with verbose output
pytest -v tests/

# Run with coverage
pytest --cov=custom_components tests/

# Local HA dev environment
docker-compose -f dev/docker-compose.yaml up
```

## Architecture

```
custom_components/power_saver/
├── __init__.py           # Entry point, platform setup
├── manifest.json         # HA integration metadata (version: 3.0.0)
├── const.py              # Constants and config keys
├── config_flow.py        # UI configuration + multi-step options flow
├── coordinator.py        # DataUpdateCoordinator, fetches prices, runs scheduler
├── scheduler.py          # Pure scheduling algorithm (no HA deps, independently testable)
├── nordpool_adapter.py   # Abstracts HACS vs native Nord Pool
├── sensor.py             # Status + diagnostic sensors (7 sensors)
├── binary_sensor.py      # Emergency mode binary sensor
├── strings.json          # English source strings (blueprint only)
└── translations/
    ├── en.json           # English runtime translations (REQUIRED)
    └── sv.json           # Swedish translations
```

### Key Components

- **scheduler.py** — Pure functions, no HA imports. Two strategies: `build_lowest_price_schedule()` (per-period optimization) and `build_minimum_runtime_schedule()` (rolling-window slot selection). `build_schedule()` is a thin dispatcher. Testable in isolation.
- **coordinator.py** — `PowerSaverCoordinator` extends `DataUpdateCoordinator`. Fetches Nord Pool data every 15 min, computes the schedule once and locks it (recomputes only when new prices arrive, settings change, or schedule expires). Controls target entities on state changes. Persists `locked_schedule` and `last_on_time`.
- **nordpool_adapter.py** — Auto-detects HACS (`raw_today` attribute) vs native HA Nord Pool. Normalizes both to `[{start, end, value}]` format.
- **config_flow.py** — Strategy-aware setup: step 1 (sensor + name + strategy) → step 2 (strategy-specific required settings). Options flow: step 1 (strategy + sensor) → step 2 (strategy settings) → step 3 (advanced: mode, thresholds, exclusion, entities). ConfigEntry VERSION = 3.

### Scheduling Strategies

**Lowest Price**: Activate cheapest N hours within fixed time periods. Supports full-day (period_from == period_to) or custom periods (e.g., 22:00→06:00). Per-period independent quota. Optional min_consecutive_hours constraint.

**Minimum Runtime**: Ensure device runs at least `min_hours_on` within each `rolling_window` of hours. UI exposes `rolling_window` (total window size); coordinator derives `max_hours_off = rolling_window - min_hours_on` for the scheduler. Selects cheapest individual 15-minute slots (scattered) unless `min_consecutive_hours` is set. Tracks `last_on_time` persistently. On first run (no history), schedules within a full `max_hours_off` window.

### Data Flow

1. Coordinator fetches prices from Nord Pool (via adapter)
2. On first run, new prices, or schedule expiry: `scheduler.build_schedule()` computes and locks the schedule
3. On subsequent refreshes: locked schedule is reused without recomputation
4. `scheduler.find_current_slot()` determines current state from the locked schedule
5. Coordinator controls target entities (switch/input_boolean/light)
6. Sensors read from `coordinator.data` (a `PowerSaverData` dataclass)

## Code Style & Patterns

- **Async/await** throughout — all HA operations are async
- **Type hints** with `from __future__ import annotations`
- **Logging** — module-level `_LOGGER = logging.getLogger(__name__)`
- **CoordinatorEntity pattern** — all entities inherit from `CoordinatorEntity[PowerSaverCoordinator]`
- **Entity naming** — `_attr_has_entity_name = True` + `_attr_translation_key` (no hardcoded names)
- **Device grouping** — entities grouped under a logical device via `DeviceInfo`
- **Optional config fields** — `None` means disabled, handled with `vol.Optional`
- **Error handling** — defensive parsing with try/except and `_LOGGER.warning()`

## Testing

- **pytest** with `asyncio_mode = auto` (see [pytest.ini](pytest.ini))
- **pytest-homeassistant-custom-component** provides HA test fixtures

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jyourstone/power_saver](https://github.com/jyourstone/power_saver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
