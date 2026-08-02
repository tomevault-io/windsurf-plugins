---
trigger: always_on
description: This is a Home Assistant custom component (`custom_components/ooler/`) that controls Ooler sleep system devices over Bluetooth Low Energy (BLE). It depends on the `ooler_ble_client` Python library (separate repo) for all BLE communication.
---

# Ooler Integration — Context for Claude Code

## Architecture

This is a Home Assistant custom component (`custom_components/ooler/`) that controls Ooler sleep system devices over Bluetooth Low Energy (BLE). It depends on the `ooler_ble_client` Python library (separate repo) for all BLE communication.

### Files

- `__init__.py` — Entry setup/unload. Creates `OolerCoordinator`, forwards platforms, registers/unregisters services. ~20 lines.
- `coordinator.py` — `OolerCoordinator` class. Manages BLE connection lifecycle, reconnection, polling, temperature unit sync, sleep schedule state, clock sync, and saved schedule persistence.
- `entity.py` — `OolerEntity` base class. Provides `device_info`, `available`, and listener registration shared by all entity platforms.
- `climate.py` — Climate entity (thermostat control: power, temperature, fan mode).
- `sensor.py` — Two sensors: Water level (%) and Schedule Tonight (tonight's sleep schedule summary with detailed attributes).
- `select.py` — Saved Schedule select entity. Lets users pick from named saved schedules to load onto the device.
- `switch.py` — Three switches: Cleaning (UV light), Sleep Schedule (toggle active schedule on/off with caching), and Bluetooth Connection (enable/disable auto-connect).
- `services.py` — Service handlers: `get_schedule`, `set_schedule`, `save_schedule`, `load_schedule`, `delete_schedule`, `clean_service`. Supports both device_id and entity_id targeting.
- `services.yaml` — Service definitions for the UI.
- `config_flow.py` — Bluetooth discovery, GATT connection verification, reconfigure flow.
- `diagnostics.py` — Diagnostics platform including connection health metrics (subscription mismatches, forced reconnects).
- `const.py` — Constants and logger.

### Library boundary

The integration never touches BLE/GATT directly. All BLE operations go through `ooler_ble_client`:

- `client.connect()` / `client.stop()` — connection lifecycle
- `client.set_power()`, `client.set_temperature()`, `client.set_mode()`, `client.set_clean()` — device commands
- `client.set_temperature_unit()` — toggle device display unit
- `client.async_poll()` — read all characteristics (also runs poll/state consistency detection)
- `client.register_callback()` — state change notifications
- `client.register_connection_event_callback()` — connection event notifications (mismatch, recovery, forced reconnect)
- `client.set_ble_device()` — update the BLEDevice for proxy routing
- `client.read_sleep_schedule()` — read schedule from device (called once on connect)
- `client.set_sleep_schedule()` / `client.clear_sleep_schedule()` — write/clear schedule
- `client.sync_clock()` — sync device clock to HA timezone
- `client.state` — `OolerBLEState` with current device state
- `client.sleep_schedule` — `OolerSleepSchedule` with current schedule (read on connect only)
- `client.is_connected` — connection status

The library handles GATT retries, `establish_connection()` via `bleak_retry_connector`, notification subscriptions, poll/state consistency detection, and `_connect_lock` serialization internally.

## Connection management

### Reconnection paths

Three mechanisms ensure reconnection when `connection_enabled` is `True`:

1. **BLE advertisement callback** (`_async_update_ble`) — fires when HA's scanner sees the device. Updates the BLEDevice and schedules a connect if disconnected.
2. **Library disconnect callback** (`_async_on_state_change`) — fires immediately when the connection drops. Schedules a staggered reconnect.
3. **Periodic timer** (`_async_reconnect_check`) — fires every 60 seconds. Refreshes BLEDevice from HA cache and attempts reconnect if disconnected.

### Connection deduplication

- `_schedule_connect()` checks `_connect_task.done()` to prevent duplicate tasks.
- `async_ensure_connected()` awaits any in-flight `_connect_task` before attempting its own connect.

### connection_enabled flag

- Set to `False` only by: the connection switch's `async_turn_off()` and `async_stop()` (teardown).
- Set to `True` only by: the connection switch's `async_turn_on()` and coordinator `__init__`.
- All reconnection paths check this flag before attempting to connect.
- Accidental disconnects leave the flag `True`, so auto-reconnect handles them.

### Stagger delay

Reconnect delay is deterministic per device (derived from MAC address hash, 0.5-2s). Only applies to auto-reconnects (`stagger=True`), not initial connects or manual switch toggles.

### Unload ordering

`async_stop()` sets `connection_enabled = False` before calling `client.stop()`. This prevents the disconnect callback from scheduling a reconnect on the old coordinator during integration reload.

`async_unload_entry()` calls `async_stop()` before `async_unload_platforms()` to give the BLE proxy time to tear down old GATT subscriptions before the new coordinator subscribes.

### BLE proxy resilience


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PostLogical/ooler](https://github.com/PostLogical/ooler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
