---
trigger: always_on
description: Python library (`tesla_fleet_api`) providing async interfaces for Tesla Fleet API, Teslemetry, and Tessie services, plus BLE communication. Published to PyPI as `tesla-fleet-api`.
---

## Project Overview

Python library (`tesla_fleet_api`) providing async interfaces for Tesla Fleet API, Teslemetry, and Tessie services, plus BLE communication. Published to PyPI as `tesla-fleet-api`.

## Development Commands

```bash
# Install dependencies
uv sync

# Type checking (strict mode)
uv run pyright tesla_fleet_api

# Linting
uv run ruff check tesla_fleet_api
uv run ruff format tesla_fleet_api

# Tests
uv run pytest tests
```

Tests live in `tests/` and use `unittest.IsolatedAsyncioTestCase` (collected and
run natively by pytest — `pytest-asyncio` is not required).

BLE command tests over a mocked transport build on `tests/ble_mocked_transport.py`
(`MockedBleTransportTestCase`): it patches `VehicleBluetooth._send` and
pre-marks both signed-command sessions ready, so a test drives any inherited
`Commands` method with no real BLE/GATT connection and asserts on the signed
`RoutableMessage` built (`decrypt_sent_command`) and on canned replies
(`vcsec_ok_reply`/`infotainment_action_ok_reply`/`infotainment_vehicle_data_reply`).
See `tests/test_ble_mocked_commands.py` for worked examples.

## API References

- Tesla Fleet: https://developer.tesla.com/docs/fleet-api/endpoints/vehicle-endpoints
- Tessie: https://developer.tessie.com/llms.txt
- Teslemetry: http://api.teslemetry.com/openapi.yaml

## Architecture

### Class Hierarchy

Three API client classes all inherit from `TeslaFleetApi`:

```
Tesla (base - tesla/tesla.py)
  └── TeslaFleetApi (tesla/fleet.py) - core HTTP client with _request(), access_token handling
        ├── TeslaFleetOAuth (tesla/oauth.py) - adds OAuth flow (login URL, token refresh)
        ├── Teslemetry (teslemetry/teslemetry.py) - fixed server, Teslemetry-specific endpoints
        └── Tessie (tessie/tessie.py) - fixed server, Tessie-specific endpoints
```

`Tesla` base holds EC key management for signed commands. `TeslaFleetApi` provides the `_request()` method used by all submodules.

### Vehicle Command Layers

Vehicle commands have three implementations sharing the same method signatures, selected by how you create the vehicle:

```
Vehicle (vehicle/vehicle.py) - base with VIN and model detection
  └── VehicleFleet (vehicle/fleet.py) - REST API commands (unsigned)
        └── VehicleSigned (vehicle/signed.py) - signed command protocol via Fleet API
Commands (vehicle/commands.py) - protobuf-based signed command implementation (ABC)
  └── VehicleSigned - multiple inheritance: Commands + VehicleFleet
  └── VehicleBluetooth (vehicle/bluetooth.py) - BLE transport for signed commands
```

`VehicleSigned` uses multiple inheritance: `Commands` for signed command logic, `VehicleFleet` for data endpoints and fallback.

`Router` (router.py) is an entity-agnostic composition wrapper (not part of the inheritance chain) that chains an ordered list of two-or-more backends sharing a common method surface and dispatches each method call down the chain with automatic per-command failover: it tries the first backend that has the method and, on any exception except `BluetoothUnconfirmedCommand`, retries the same call on the next backend that has it, returning the first success (raising the last error only if every applicable backend fails, `AttributeError` only if none has the method). Non-callable attributes resolve to the first backend that has them. The constructor is `Router(primary, secondary, *more_backends, health=None)` — the two-argument form is fully backward compatible. The health check (`bool` | sync callable | async callable returning `bool`; omitted = attempt primary, fail over on exception with no probe) gates **only the primary** (the first backend); the rest of the chain is reached purely through per-command failover — there is deliberately no per-backend health matrix. Note the double-execution caveat: a non-idempotent command that fails mid-flight can be re-run on the next backend, except for `BluetoothUnconfirmedCommand`, which propagates without replay.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Teslemetry/python-tesla-fleet-api](https://github.com/Teslemetry/python-tesla-fleet-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
