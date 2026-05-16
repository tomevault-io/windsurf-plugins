---
trigger: always_on
description: Arduino/PlatformIO library (C++17) for connecting BLE gamepads to ESP32 boards. The core design goal is **simplicity of use**: users instantiate a controller, call `begin()`, and either poll with `read()` or register callbacks. All BLE complexity is hidden behind this surface.
---

# BLE-Gamepad-Client — Developer Guide

## Project overview

Arduino/PlatformIO library (C++17) for connecting BLE gamepads to ESP32 boards. The core design goal is **simplicity of use**: users instantiate a controller, call `begin()`, and either poll with `read()` or register callbacks. All BLE complexity is hidden behind this surface.

- **Platform**: ESP32 only (`architectures=esp32`), Arduino framework
- **Dependency**: `h2zero/NimBLE-Arduino ^2.3.2`
- **License**: Apache-2.0

## Build & CI

Build is tested exclusively with PlatformIO. No local build script exists; use the CI workflow as a reference.

```bash
# Minimal platformio.ini for development
[env:esp32dev]
platform = espressif32
framework = arduino
board = esp32dev
build_unflags = -std=gnu++14
```

CI (`build.yml`) tests five examples across three boards: `esp32dev`, `esp32s3`, `esp32c3`. Docs are built with MkDocs + Material (`mkdocs==1.6.1`, `mkdocs-material==9.7.0`). Arduino Lint runs on every PR.

## Architecture

### Component map

```
BLEGamepadClient          static façade / entry point, initializes NimBLE
├── BLEAutoScan           FreeRTOS task: manages scan lifecycle (high-duty → low-duty)
├── BLEControllerRegistry manages registered controllers, owns NimBLE client callbacks
│   └── ClientCallbacksImpl  NimBLE callbacks → internal ClientEvent queue
└── BLEUserCallbackRunner drains userCallbackQueue, fires user-facing callbacks

BLEAbstractController     connection state machine, address allocation (lock-free CAS)
└── BLEBaseController<T>  CRTP layer: typed onConnecting/onConnected/onDisconnected callbacks
    └── XboxController / SteamController
        ├── BLEValueReceiver<XboxControlsState>   subscribes to HID notifications
        ├── BLEValueReceiver<XboxBatteryState>
        └── BLEValueWriter<XboxVibrationsCommand> writes HID output reports
```

### Threading model

There are four FreeRTOS tasks plus the NimBLE stack's own tasks:

| Task | Purpose |
|---|---|
| `_autoScanTask` | Receives task notifications, starts/stops scans |
| `_clientEventConsumerTask` | Serialises NimBLE client events (pinned to `CONFIG_BT_NIMBLE_PINNED_TO_CORE`) |
| `BLEUserCallbackRunner` task | Drains `_userCallbackQueue`, calls all user-facing callbacks |
| `_callbackTask` (one per `BLEValueReceiver`) | Fires `onValueChanged` callbacks off the BLE notify thread |

User callbacks are always invoked from the `BLEUserCallbackRunner` task or a `BLEValueReceiver` callback task — never directly from a NimBLE callback. This means user callback code does not need to be ISR-safe, but it should not block for long.

### Connection sequence

`begin()` → registered → auto-scan starts → device discovered → `tryAllocate()` (lock-free) → NimBLE client created → `connect()` → `onConnect` (NimBLE) → `secureConnection()` → `onAuthenticationComplete` bonded → `hidInit()` + controller `init()` → `markConnected()` → `onConnected` user callback.

On disconnect: `onDisconnect` (NimBLE) → `deinit()` → `tryDeallocate()` → `deleteClient()` → `onDisconnected` user callback → auto-scan restarts.

### Auto-scan

Scans in two phases after `begin()`:
1. **High-duty** (default 60 s): aggressive window/interval (10 ms / 10 ms), active scan
2. **Low-duty** (default 240 s): power-efficient (1280 ms / 15 ms), passive scan

`BLEAutoScan::notify()` restarts scanning manually if the scan window expired before all slots filled.

## Conventions

### Naming

- Classes: `PascalCase`, infrastructure prefixed `BLE`, user-facing types are descriptive (`XboxController`, `XboxControlsState`)
- Private members: `_camelCase`
- Public methods: `camelCase`
- Config overrides: `CONFIG_BT_BLEGC_*` preprocessor defines (see `src/config.h`)
- Utility namespace: `blegc::` (BLE GATT UUIDs, appearance constants, helper functions)
- Logging macros: `BLEGC_LOGV/D/I/W/E`

### Value types (state & command structs)

All state and command structs extend `BLEBaseValue`:

- **State structs** (`*ControlsState`, `*BatteryState`) implement `decode(uint8_t[], size_t)` → `BLEDecodeResult` and equality operators
- **Command structs** (`*VibrationsCommand`) implement `encode(size_t&, uint8_t[], size_t)` → `BLEEncodeResult`
- Analog axes / triggers are normalised to `float`: sticks `[-1.0, 1.0]`, triggers/battery `[0.0, 1.0]`
- `controllerAddress` (peer `NimBLEAddress`) is set by `BLEValueReceiver` on subscribe

### Headers

- `#pragma once` everywhere (no include guards)
- All public types are re-exported through `src/BLEGamepadClient.h`, which is the single include for users

## Adding a new controller

1. Create `src/<name>/` directory.
2. Add a `*ControlsState` struct extending `BLEBaseValue` — implement `decode()`, `operator==`, `operator!=`.
3. Add a `*Controller` class extending `BLEBaseController<ControllerType>` and `BLEValueReceiver<ControlsState>`:
   - Override `isSupported(const NimBLEAdvertisedDevice*)` — identify device by name / appearance / manufacturer ID
   - Override `init()` — find characteristics with `blegc::findNotifiableCharacteristic()`, call `BLEValueReceiver<T>::init()`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tbekas/BLE-Gamepad-Client](https://github.com/tbekas/BLE-Gamepad-Client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
