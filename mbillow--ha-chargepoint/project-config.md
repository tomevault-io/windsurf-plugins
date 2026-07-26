---
trigger: always_on
description: This file provides guidance for AI assistants working on the `ha-chargepoint` repository.
---

# CLAUDE.md

This file provides guidance for AI assistants working on the `ha-chargepoint` repository.

## What This Is

A Home Assistant custom component integration for ChargePoint EV charging stations. It is a cloud-polling integration (`iot_class: cloud_polling`) distributed via HACS. The integration wraps the [`python-chargepoint`](https://github.com/mbillow/python-chargepoint) library, which is **maintained by the same author as this repo**.

Domain: `chargepoint`.

## Repository Layout

```
custom_components/chargepoint/   # All integration source code
    __init__.py                  # setup, DataUpdateCoordinator, base entity classes
    const.py                     # all constants and coordinator data keys
    config_flow.py               # config + options UI flows
    manifest.json                # HA integration metadata, dependency declarations
    binary_sensor.py             # public station availability
    button.py                    # start/stop charging, restart charger
    number.py                    # LED brightness
    select.py                    # amperage limit
    sensor.py                    # account balance, session metrics
    switch.py                    # charging schedule enable/disable
    time.py                      # schedule start/end times
    diagnostics.py               # anonymized debug data export
    translations/en.json         # UI strings
tests/
    conftest.py                  # all shared fixtures and mock factories
    test_init.py
    test_binary_sensor.py
    test_button.py
    test_config_flow.py
    test_number.py
    test_select.py
    test_sensor.py
```

## Development Workflow

### Python Version

The canonical Python version is defined in `.github/workflows/combined.yaml` under `python-version:` in the `lint-and-test` job. There is no `.python-version` file — always read the workflow to get the current value before doing anything else.

**Always use the exact version from the workflow.** Do not use a generic `python3` invocation — it may resolve to a different version. For example, if the workflow specifies `3.13`, use `python3.13`.

### Installing Dependencies

**Always create a venv with the exact Python version from the workflow before installing anything.** Never install dependencies globally or skip venv creation because packages appear to be available already.

```bash
python3.X -m venv .venv   # X = version from combined.yaml
source .venv/bin/activate
pip install -r requirements_test.txt
```

This installs all test, lint, type-check, and formatting tools plus `python-chargepoint`.

### Running Checks

**All checks are run via pre-commit from within the activated venv. This is the single source of truth.**

```bash
source .venv/bin/activate
pre-commit run --all-files
```

This runs in order: `trailing-whitespace`, `end-of-file-fixer`, `check-yaml`, `check-added-large-files`, `black`, `flake8`, `isort`, `mypy`, `pyright`, `pytest`.

Do not run `pytest`, `mypy`, `black`, or any other tool individually — always use `pre-commit run --all-files`. **Do not commit unless this passes cleanly.**

### Local HA Instance (Optional)

A `docker-compose.yaml` is provided for running a local Home Assistant instance with the integration mounted live:

```bash
docker compose up
```

HA will be available at `http://127.0.0.1:8123`. The integration source is bind-mounted from `./custom_components/chargepoint`.

## Code Patterns

### Base Classes

Every entity inherits from one of two base classes defined in `__init__.py`:

- `ChargePointEntity(CoordinatorEntity)` — for account-level entities (e.g. account balance)
- `ChargePointChargerEntity(CoordinatorEntity)` — for per-charger entities

Platform entities then mix in the HA platform base, e.g.:

```python
class ChargePointChargerSensorEntity(SensorEntity, ChargePointChargerEntity):
    ...
```

### Entity Descriptions

Use frozen dataclasses extending the platform's `*EntityDescription`:

```python
@dataclass(frozen=True)
class ChargePointSensorEntityDescription(SensorEntityDescription):
    name_suffix: str = ""
    value: Callable[..., StateType] = field(default=lambda _: None)
```

### Unique IDs

- Account-level: `{user_id}_{description.key}`
- Charger-level: `{charger_id}_{description.key}`
- Public station: `public_{station_id}_{description.key}` — the `public_` prefix is significant; it drives entity ID migration logic in `_migrate_public_entity_ids()`.

Do not change the unique ID format for existing entities — it will break entity registry entries for existing users.

### Coordinator Data Keys

All coordinator data keys are constants in `const.py`:

- `ACCT_INFO` — account object
- `ACCT_CRG_STATUS` — user charging status
- `ACCT_SESSION` — active charging session (may be `None`)
- `ACCT_HOME_CRGS` — `dict[charger_id, dict]` with sub-keys: `ACCT_CHARGER_STATUS`, `ACCT_CHARGER_TECH_INFO`, `ACCT_CHARGER_CONFIG`, `ACCT_CHARGER_SCHEDULE`
- `ACCT_PUBLIC_STATIONS` — `dict[station_id, StationInfo]`

### Adding a New Entity or Platform

1. Add the platform to `PLATFORMS` in `const.py` and add the file.
2. Define an entity description dataclass and a list of descriptions.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mbillow/ha-chargepoint](https://github.com/mbillow/ha-chargepoint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
