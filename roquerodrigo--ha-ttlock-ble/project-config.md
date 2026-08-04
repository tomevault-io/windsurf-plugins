---
trigger: always_on
description: Guidance for Claude Code (claude.ai/code) agents working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) agents working in this repository.

## Always read `CODE_STYLE.md` first

Before creating, renaming or restructuring any file/class/function, **read [`CODE_STYLE.md`](./CODE_STYLE.md)**. It is the single source of truth for conventions: language, file organisation, naming, typing, properties vs `__init__`, imports, docstrings, comments, coordinator pattern, diagnostics layout, translations, lint workflow.

For user-facing topics (what's included, how to fork, rename steps, layout diagram, useful commands, CI list), see [`README.md`](./README.md).

This file deliberately avoids restating those rules — it only adds:

1. The verification workflow agents must run after every change.
2. The architectural reasoning that is not obvious from `CODE_STYLE.md` alone.

## Verification workflow

**After every code change, always run lint then tests, in that order, before declaring the task done:**

```bash
uv run ruff format --check . && uv run ruff check . && uv run mypy custom_components/ttlock_ble && uv run pytest
```

- `ruff format --check .`, `ruff check .` and `mypy custom_components/ttlock_ble` enforce formatting, linting and strict typing. Fix any failure and re-run before moving on.
- `pytest` runs with the `--cov` flags (configured in `pyproject.toml`) and enforces the coverage gate. Both ruff and mypy configuration also live in `pyproject.toml`.

Both gates mirror CI (`.github/workflows/ci.yml`). Skip this only when the change literally cannot affect lint or tests (e.g., README-only edits).

## Bumping the Home Assistant version

The Home Assistant version is pinned in two places and **must be updated together**, otherwise CI, HACS and the test harness drift apart:

1. `pyproject.toml` — the `dependency-groups.dev` list pins both `homeassistant==<X.Y.Z>` (runtime/CI lint + mypy) and `pytest-homeassistant-custom-component==<matching release>` (the test harness ships its own pinned `homeassistant`; the two pins must come from the same HA release, otherwise lint and tests resolve different cores).
2. `hacs.json` — `"homeassistant": "<X.Y.Z>"` (minimum HA core enforced by HACS; may lag behind the dev-group pin).

Verify the pairing on PyPI before committing: the `requires_dist` of `pytest-homeassistant-custom-component` must list the same `homeassistant==<X.Y.Z>` you pinned in `pyproject.toml`. Run `uv sync` after bumping so `uv.lock` picks up the new pins.

## The `ttlock-ble` SDK pin

This integration wraps the sibling repo [`ttlock-ble`](https://github.com/roquerodrigo/ttlock-ble), which owns all BLE protocol/crypto logic and the cloud login client. The pin lives in **two places that can drift**:

- `custom_components/ttlock_ble/manifest.json` → `requirements: ["ttlock-ble==<version>"]` — this is what HACS/HA actually installs for end users.
- `pyproject.toml` → `dependency-groups.dev` → `"ttlock-ble==<version>"` — this is what lint/mypy/pytest run against locally and in CI.

If these two pins diverge, CI is green against a different SDK version than what ships to users. A breaking change in the SDK's public API (`TTLockCloud`, `TTLockClient`, `VirtualKey`, `CloudError`, the `disconnected_callback` signature) requires bumping both pins together, then re-running lint + tests here — the SDK repo's own release does not by itself update anything on this side.

## Architecture

The integration follows the HA `DataUpdateCoordinator` pattern:

```
config_flow.py   → cloud-bootstraps credentials, requests 2FA when needed,
                    pulls the per-lock VirtualKeys, creates the ConfigEntry
manual_key.py    → builds a VirtualKey from a hand-entered key, for locks
                    that were never in a TTLock account
__init__.py      → instantiates one TtlockBleConnection per lock and a
                    DataUpdateCoordinator, performs the first refresh
connection.py    → owns the long-lived BLE session, reconnect loop,
                    cooldown, and push-event dispatch
advertisement.py → decodes lock state + battery from the advertisements
                    HA's bluetooth manager already receives, no connection
coordinator.py   → polls every scan_interval seconds via each connection,
                    and publishes the advertised state as it arrives
lock.py          → LockEntity backed by the BLE connection
sensor.py        → BatterySensor backed by the same poll + push events
binary_sensor.py → connectivity BinarySensorEntity reflecting live BLE link state
event.py         → EventEntity that surfaces decoded LockEvent pushes
```

### Entry typing

`data/` is a package, one class per file, re-exported from `data/__init__.py`. `data/__init__.py` defines `TtlockBleConfigEntry = ConfigEntry[TtlockBleData]`; `data/runtime.py` defines the `TtlockBleData(keys, virtual_keys, connections, coordinator, bluetooth_unsubs)` dataclass. State lives on `entry.runtime_data` (auto-discarded on unload), never on `hass.data`.

### Config flow surface

`config_flow.py` implements the user-facing steps, sharing one module-level `_credentials_schema` builder for the credential ones and one `_manual_key_schema` for the manual one:

- `async_step_user` — a menu, nothing else: the entry can come from a cloud account or from a key entered by hand.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [roquerodrigo/ha-ttlock-ble](https://github.com/roquerodrigo/ha-ttlock-ble) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-02 -->
