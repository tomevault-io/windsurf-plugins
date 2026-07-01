---
trigger: always_on
description: Guidelines for AI assistants working on **ruuvitag-sensor** — a Python library for communicating with RuuviTag BLE sensors and Ruuvi Air devices, decoding measurement data from BLE broadcasts and GATT history.
---

# AI Agents Configuration

Guidelines for AI assistants working on **ruuvitag-sensor** — a Python library for communicating with RuuviTag BLE sensors and Ruuvi Air devices, decoding measurement data from BLE broadcasts and GATT history.

**Rule:** Do not create new documentation files. Update existing docs (`README.md`, `developer_notes.md`, this file). If a new file seems necessary, ask first.

---

## Project Overview

- **Package:** `ruuvitag_sensor` (v4.0.0, Production/Stable)
- **Language:** Python 3.10+ (supports 3.10–3.14)
- **Build:** setuptools + wheel (`pyproject.toml`)
- **Dependencies:** `bleak` (BLE), `reactivex` (RxPY), `ptyprocess` (Linux only)
- **Dev deps:** `pytest`, `pytest-asyncio`, `ruff`, `mypy`
- **License:** MIT
- **Docs:** `README.md` (usage), `developer_notes.md` (dev guide, BLE internals, release)

---

## Architecture

```
ruuvitag_sensor/
├── ruuvi.py              # Main API: RuuviTagSensor (static methods)
├── ruuvitag.py           # RuuviTag/RuuviTagAsync (single-sensor class)
├── ruuvi_rx.py           # ReactiveX wrapper (RuuviTagReactive)
├── ruuvi_types.py        # TypedDict definitions for sensor data
├── data_formats.py       # BLE advertisement parsing & format detection
├── decoder.py            # get_decoder() router, parse_mac(), re-exports
├── log.py                # Logging: enable_console(), file logging
├── __main__.py           # CLI entry point
├── decoders/             # One module per data format
│   ├── __init__.py       # Exports all decoder classes
│   ├── url_decoder.py    # DF2/DF4 (deprecated, Eddystone URL)
│   ├── df3_decoder.py    # DF3 (deprecated, simple raw)
│   ├── df5_decoder.py    # DF5 (RuuviTag primary format)
│   ├── df6_decoder.py    # DF6 (Ruuvi Air broadcast)
│   ├── dfe1_decoder.py   # DFE1 (Ruuvi Air extended, preferred over DF6)
│   ├── history_decoder.py      # RuuviTag history (single-record per entry)
│   └── air_history_decoder.py  # Ruuvi Air history (multi-record batches)
└── adapters/             # BLE communication backends
    ├── __init__.py       # Abstract base classes, get_ble_adapter()
    ├── bleak_ble.py      # Bleak (default, async, cross-platform)
    ├── nix_hci.py        # BlueZ hcitool/hcidump (Linux, sync, legacy)
    ├── bleson.py         # Bleson (experimental, not recommended)
    ├── nix_hci_file.py   # File-based BlueZ emulation (testing)
    ├── dummy.py          # Hardcoded test data (CI/offline)
    ├── utils.py          # Adapter utilities
    └── development/      # Dev tools (dev_bleak_scanner.py)

tests/
├── test_data_formats.py              # Format detection tests
├── test_ruuvitag_sensor.py           # Sync API tests (mocked adapter)
├── test_ruuvitag_sensor_async.py     # Async API tests
├── test_bleak_history_notification.py
├── hcidump-2.x.txt, hcidump-3.x.txt # Test fixture files
├── decoders/                         # Per-decoder test modules
│   ├── test_url_decoder.py, test_df3_decoder.py, test_df5_decoder.py
│   ├── test_df6_decoder.py, test_dfe1_decoder.py
│   ├── test_history_decoder.py, test_air_history_decoder.py
│   └── test_get_decoder.py           # get_decoder() routing tests
└── integration/                      # Real BLE integration tests
    ├── test_integration_bleak_get_data.py
    ├── test_integration_bleak_history.py
    └── test_integration_bleak_air_history.py

examples/                  # Usage examples (async, sync, MQTT, InfluxDB, HTTP servers)
```

---

## Data Flow

1. **BLE adapter** receives raw advertisement bytes
2. **`DataFormats.convert_data(raw_hex)`** in `data_formats.py`:
   - Dechunks BLE AD structure (length + type + data chunks)
   - Identifies Ruuvi manufacturer ID `9904` + format byte
   - Returns `(data_format, payload)` tuple
3. **`get_decoder(data_format)`** in `decoder.py`:
   - Uses `match` statement to return the correct decoder instance
   - Format IDs: `int` for most (2,3,4,5,6), `str` for E1 (`"E1"`)
4. **Decoder's `decode_data(payload)`** returns `SensorData` dict
5. **`parse_mac(data_format, payload_mac)`** extracts MAC (only DF5 has MAC in payload)

---

## Data Formats

| Format | Decoder | Device | Status | Key Fields |
|--------|---------|--------|--------|------------|
| 2, 4 | `UrlDecoder` | RuuviTag | Deprecated | temp, humidity, pressure, identifier |
| 3 | `Df3Decoder` | RuuviTag | Deprecated | + acceleration (x/y/z), battery |
| **5** | `Df5Decoder` | **RuuviTag** | **Primary** | + tx_power, movement_counter, sequence, mac, rssi |
| **6** | `Df6Decoder` | **Ruuvi Air** | Active | + pm2_5, co2, voc, nox, luminosity |
| **E1** | `DfE1Decoder` | **Ruuvi Air** | Active | + pm1, pm4, pm10 (extends DF6, preferred) |
| History | `HistoryDecoder` | RuuviTag | Active | Single measurement per entry + timestamp |
| Air History | `AirHistoryDecoder` | Ruuvi Air | Active | Multi-record batches + timestamps |

**Important:** RuuviTag and Ruuvi Air use **different history protocols** — different NUS services, different data structures, different decoders.

---

## BLE Adapters

| Adapter | Class | Async | Platform | Selection |
|---------|-------|-------|----------|-----------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ttu/ruuvitag-sensor](https://github.com/ttu/ruuvitag-sensor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
