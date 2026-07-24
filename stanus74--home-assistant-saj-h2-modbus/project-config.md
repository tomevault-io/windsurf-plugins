---
trigger: always_on
description: **Status:** Updated Jan 26 2026 (v2.8.1) | **Project:** Unofficial Home Assistant integration for SAJ H2 inverters (reverse-engineered Modbus registers)
---

# AI Agent Instructions for SAJ H2 Modbus Integration

**Status:** Updated Jan 26 2026 (v2.8.1) | **Project:** Unofficial Home Assistant integration for SAJ H2 inverters (reverse-engineered Modbus registers)

## Quick Start for AI Agents

**Before modifying code, understand (see also `docs/architecture_overview.md`):**

1.  **3-tier polling architecture** with separate asyncio locks (`_slow_lock`, `_fast_lock`, `_ultra_fast_lock`) plus a dedicated `_write_lock`.
    *   **Standard (60s)**: Sequential execution of all readers using a unified slow lock.
    *   **Fast (10s)**: High-frequency sensors only; disabled when ultra-fast is enabled.
    *   **Ultra-Fast (1s)**: MQTT-only, fire-and-forget, skipped during write operations.
2.  **Configuration-driven** register parsing - add sensors via `_DATA_READ_CONFIG` dicts.
3.  **Command queue pattern** for all writes - `ChargeSettingHandler` uses `asyncio.Queue` to serialize and prioritize writes over reads.
4.  **Strategy pattern** for MQTT - prefers HA MQTT, falls back to paho-mqtt.
5.  **Connection caching** in `ModbusConnectionManager` (60s TTL) with serialized cache access.
6.  **Circuit breaker** shared across Modbus reads/connect and MQTT publishing.


## Project Overview

**Context:** Unofficial community integration for reading SAJ H2 inverters (8kW-10kW) via Modbus TCP.

**Key Files:**
- [hub.py](../custom_components/saj_h2_modbus/hub.py) - DataUpdateCoordinator, polling orchestration.
- [modbus_readers.py](../custom_components/saj_h2_modbus/modbus_readers.py) - Configuration-driven register parsing.
- [modbus_utils.py](../custom_components/saj_h2_modbus/modbus_utils.py) - Low-level Modbus TCP, retry logic, connection caching.
- [charge_control.py](../custom_components/saj_h2_modbus/charge_control.py) - Async command queue for write operations.
- [services.py](../custom_components/saj_h2_modbus/services.py) - ModbusConnectionManager, MqttPublisher.
- [const.py](../custom_components/saj_h2_modbus/const.py) - Sensor definitions.
- [switch.py](../custom_components/saj_h2_modbus/switch.py) - Charging/Discharging and Passive Mode switches.

## Architecture & Data Flow

### Multi-Level Polling System

1.  **Standard (60s)**: Reads all data. Uses `_slow_lock` and executes reader groups sequentially.
2.  **Fast (10s)**: Reads `FAST_POLL_SENSORS` only. Uses `_fast_lock`. Updates HA entities. Disabled when ultra-fast is enabled.
3.  **Ultra-Fast (1s)**: Reads `FAST_POLL_SENSORS`. Uses `_ultra_fast_lock`. Publishes to MQTT only. **Skipped if a write operation is in progress.**

### Component Responsibilities

**SAJModbusHub**:
- Orchestrates polling loops.
- Manages `_merge_locks` for registers 0x3604/0x3605.
- Holds `inverter_data` (source of truth).

**ChargeSettingHandler**:
- Queues all commands (`CommandType`).
- `process_queue` runs in background task.
- Uses `_write_register_with_backoff` for reliability.
- **Optimistic UI**: Updates `inverter_data` immediately after a successful Modbus write to give instant feedback.

**ModbusReader**:
- `_read_modbus_data()`: Returns `new_data` dict and `errors` list. Even if some registers fail, valid data is returned.
- **Configuration**: Use `_DATA_READ_CONFIG` to add new registers/sensors.

**Switch Entities**:
- **Charging/Discharging**: Controls registers 0x3604/0x3605 + AppMode (0x3647).
- **Passive Mode**: Controls `passive_charge_enable` and AppMode=3. Controlled via `PASSIVE_SWITCH_KEYS` in `switch.py`.

### 7 Functional Domains (from `/plans/FUNKTIONSBEREICHE-ANALYSE.md`)

The codebase is organized into 7 independent functional areas:

1. **Modbus Register Reading** - Data ingestion, decoding, caching
2. **HA Entity Updates** - State publishing, fast listeners, registry management
3. **MQTT Publishing** - Strategy pattern, filtering, async publishing
4. **Charge Control** - Command queue, write operations, optimization
5. **Configuration Flow** - Setup UI, options, validation
6. **Entity Optimization** - Write confirmation, optimistic updates, error recovery
7. **Performance & Lifecycle** - Lock management, startup/shutdown, resource cleanup

## Development Patterns & Conventions

### Lock Management (Critical - Do Not Skip)

**Problem:** Using a single lock for all polling serializes operations, blocking fast updates behind slow ones.

**Solution:** Use appropriate lock based on operation type:
```python
# Read high-frequency data (10s loop)
async with self._fast_lock:
    data = await modbus_readers.read_additional_modbus_data_1_part_2(client, lock)

# Read all data (60s loop)  
async with self._slow_lock:
    data = await modbus_readers.read_realtime_data(client, lock)

# Write operations (prioritized)
async with self._write_lock:
    success = await hub._write_register(address, value)
```

**Merge locks:** Registers 0x3604 (charge slots) and 0x3605 (discharge slots) are pure slot bitmasks (bit 0 = Slot 1 … bit 6 = Slot 7). Use `_merge_locks` for read-modify-write to prevent corruption. **Special Case:** When disabling charging/discharging entirely (via Switch), the register must be set to `0` (clearing all slots).

```python
async with self._merge_locks[0x3604]:
    current = await read_registers(0x3604, 1)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stanus74/home-assistant-saj-h2-modbus](https://github.com/stanus74/home-assistant-saj-h2-modbus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
