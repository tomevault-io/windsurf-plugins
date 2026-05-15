---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Run all tests
```bash
pip install -r requirements-test.txt
pytest
```

### Lint (always include tests/)
```bash
ruff check custom_components/ tests/
```

### Run a single test file
```bash
pytest tests/test_parser.py
```

### Run a single test class or function
```bash
pytest tests/test_parser.py::TestParseInfoT1Response
pytest tests/test_parser.py::TestParseBattery::test_valid_value
```

No build step is needed – this is a pure Python Home Assistant custom integration installed by copying `custom_components/oclean_ble/` into an HA config directory.

---

## Architecture

This is a **Home Assistant custom integration** (`domain: oclean_ble`) for Oclean Smart Toothbrushes. It connects via BLE every N seconds (default 300 s), reads brushing data, then disconnects. No cloud, no account.

### Data flow

```
BLE Device
  └─ establish_connection (bleak_retry_connector)
       └─ OcleanCoordinator._poll_device()
            ├─ _calibrate_time()           → write CMD 020E + timestamp via DeviceProtocol.write_char
            ├─ _read_device_info_service() → BLE DIS (model/fw/hw, cached 24 h)
            ├─ _subscribe_notifications()  → up to 4 GATT notify characteristics (per DeviceProtocol.notify_chars)
            ├─ _send_query_commands()      → CMD sequence from DeviceProtocol.query_commands
            │                                TYPE1: 0303/0202/0302/0307 all via fbb89 (SEND_BRUSH_CMD_UUID)
            │                                TYPE0: 0303/0202/0302/0308 all via fbb85 (WRITE_CHAR_UUID)
            ├─ _paginate_sessions()        → CMD 0309 via fbb85 until no new sessions (TYPE0 only)
            └─ _read_battery_and_unsubscribe()
                  → notification_handler() calls parse_notification()
                       └─ _PARSERS registry dispatches by 2-byte prefix
                            ├─ 0303 → _parse_state_response()
                            ├─ 0307 → _parse_info_t1_response()   (Type-1 / Oclean X family)
                            ├─ 0308 → _parse_info_response()      (Type-0; auto-detects extended vs. simple binary format)
                            ├─ 0202 → _handle_device_info_ack()
                            ├─ 0000 → _parse_score_t1_response()  (Type-1 enrichment: score push after session)
                            ├─ 2604 → _parse_brush_areas_t1_response() (Type-1 enrichment: per-tooth areas)
                            └─ 0340 → _parse_k3guide_response()   (K3-series real-time guidance)
  └─ OcleanDeviceData (dataclass snapshot)
       └─ CoordinatorEntity subclasses (OcleanSensor, OcleanBinarySensor, OcleanButton)
```

### Key files

| File | Role |
|------|------|
| `coordinator.py` | `OcleanCoordinator` – BLE polling, session pagination, HA long-term statistics import, stale-data persistence |
| `parser.py` | Pure functions: BLE byte payload → dict. Strategy registry `_PARSERS` keyed on 2-byte response prefix. Two binary record formats (simple 18-byte and extended 32-byte). |
| `protocol.py` | `DeviceProtocol` dataclass – per-family capability profile: `notify_chars`, `query_commands`, `supports_pagination`, `write_char`. `protocol_for_model()` maps DIS model-IDs to profiles. |
| `models.py` | `OcleanDeviceData` dataclass – typed snapshot returned by the coordinator. Field names match `DATA_*` string constants from `const.py` so sensors look up values via `getattr`. |
| `const.py` | All GATT UUIDs, BLE command bytes, response type markers, `DATA_*` / `SENSOR_*` key constants, `SCHEME_NAMES` lookup, `TOOTH_AREA_NAMES` tuple |
| `entity.py` | `OcleanEntity` base class – shared `unique_id`, `device_info`, and `available` logic |
| `sensor.py` | All sensor entities; `OcleanSensor` (generic), `OcleanBrushAreasSensor` (zone dict as attributes), `OcleanSchemeSensor` (pNum + name attribute) |
| `config_flow.py` | Config + options flow; auto-discovery via Bluetooth, manual MAC entry fallback |
| `__init__.py` | `async_setup_entry` / `async_unload_entry`; attaches/detaches a rotating file log handler (`oclean_ble.log`, 1 MB × 3) shared across all config entries |

### Parser format detection (0308 path)

`_parse_info_response()` auto-detects which binary format to use:
- **Extended (32+ bytes):** `payload[0] == 0` AND `payload[1] >= 32` AND `len(payload) >= payload[1]` → `_parse_extended_running_data_record()` – yields score, duration, 8 tooth area pressures, scheme type, pNum
- **Simple (18 bytes):** fallback → `_parse_running_data_record()` – yields timestamp, pressure, brush-head wear counter

The Type-1 path (0307) is a separate handler `_parse_info_t1_response()` with a different byte layout (device constant in bytes 0-4, timestamp bytes 5-10, brushing metric byte 13).

### Session import and deduplication

`OcleanCoordinator` persists state in HA storage (one store per MAC, keyed `oclean_ble.<mac_slug>`):
- `last_session_ts` – Unix timestamp of the newest imported session; prevents re-importing sessions across restarts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deniskie/ha-oclean-integration](https://github.com/deniskie/ha-oclean-integration) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
