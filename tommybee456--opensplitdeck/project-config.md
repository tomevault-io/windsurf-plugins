---
trigger: always_on
description: This is firmware for a **split wireless game controller** running on Seeed Xiao BLE nRF52840 boards using the **Zephyr RTOS** and **Nordic ESB (Enhanced ShockBurst)** wireless protocol. Each controller half (LEFT/RIGHT) is a separate nRF52840 board that communicates with a central dongle.
---

# OpenSplitDeck Controller Firmware - AI Coding Agent Instructions

## Project Overview
This is firmware for a **split wireless game controller** running on Seeed Xiao BLE nRF52840 boards using the **Zephyr RTOS** and **Nordic ESB (Enhanced ShockBurst)** wireless protocol. Each controller half (LEFT/RIGHT) is a separate nRF52840 board that communicates with a central dongle.

## Critical Architecture Concepts

### Controller Identity System
- **CONTROLLER_ID macro** in [main.c](src/main.c#L16) determines LEFT (1) or RIGHT (0) controller
- Controllers have distinct radio pipelines and respond only to their ID
- The `flags` byte in `esb_controller_data_t` uses bit 7 to indicate LEFT (0x80) or RIGHT (0x00)
- **Always respect this ID when modifying ESB communication or initialization code**

### Driver-Based Architecture
This firmware uses a **modular driver pattern** - each peripheral has a dedicated driver with init/update/get_data pattern:

| Driver | Header | Purpose | Key Functions |
|--------|--------|---------|---------------|
| `esb_comm_driver` | [esb_comm_driver.h](src/esb_comm_driver.h) | Wireless communication via Nordic ESB | `esb_comm_send_data()`, adaptive timing with ACK payloads |
| `analog_driver` | [analog_driver.h](src/analog_driver.h) | Thread-based ADC reads for sticks/triggers | `analog_driver_get_controller_data()`, background sampling |
| `button_driver` | [button_driver.h](src/button_driver.h) | Debounced GPIO button handling | `button_driver_read_all()`, edge detection |
| `imu_driver` | [imu_driver.h](src/imu_driver.h) | LSM6DS3 accelerometer/gyro | `imu_get_controller_data()`, complementary filter |
| `haptic_driver` | [haptic_driver.h](src/haptic_driver.h) | DRV2605 LRA motor control | `haptic_setup_external_trigger()`, GPIO trigger mode |
| `display` | [display.h](src/display.h) | SSD1306 OLED (128x32) | Thread-safe screen management |
| `controller_storage` | [controller_storage.h](src/controller_storage.h) | NVS-based persistent config | Calibration, bindings, preferences |
| `power_mgmt_driver` | [power_mgmt_driver.h](src/power_mgmt_driver.h) | Sleep/wake, button combos | Peripheral shutdown orchestration |

**Pattern**: When adding features, use the existing driver interfaces rather than direct hardware access. Drivers handle thread safety and error recovery.

### Multi-Threading Architecture
The firmware uses **three concurrent threads** coordinated via shared memory with atomic updates:

```c
Main Thread (priority 0):     ESB transmission @ ~125Hz, button reads, sleep combo detection
Trackpad Thread (priority 5): IQS7211E polling @ 10ms, haptic pulse management  
Display Thread (priority 6):  Screen rendering @ 30Hz, lockless screen switching
```

**Critical**: `trackpad_thread_heartbeat` and `display_thread_heartbeat` are monitored in main loop - frozen threads trigger warnings. Never block these threads with long sleeps.

### ESB Adaptive Timing System
The ESB driver implements **dongle-controlled transmission timing** via ACK payloads:

1. Controller sends data at adaptive rate (default 8ms)
2. Dongle responds with `ack_timing_data_t` containing `next_delay_ms`
3. Controller adjusts sleep delay based on dongle requirements
4. **Never hardcode sleep delays in main loop** - use `esb_comm_get_next_delay()`

See [main.c lines 2368-2385](src/main.c#L2368-L2385) for timing implementation. The system includes radio busy backoff and latency spike detection.

## Build System (Zephyr + West)

### Building the Firmware
```bash
# Build for specific board (from controller/ directory)
west build -b seeed_xiao_ble_nrf52840_sense

# Flash via USB bootloader (double-tap reset button)
west flash

# Clean build
west build -t clean
```

### Key Build Files
- **CMakeLists.txt**: Source file list (comment out files to disable modules)
- **prj.conf**: Kconfig options - logging levels, peripheral enables, ESB config
- **xiao_ble_nrf52840_sense.overlay**: Device tree hardware config (I2C pins, ADC channels, display)
- **boards/seeed_xiao_ble.overlay**: Board-specific variant

**Important**: When modifying prj.conf, changes take effect on next build. Common issues:
- `CONFIG_I2C_NRFX_TRANSFER_TIMEOUT=100` prevents I2C freezes
- `CONFIG_NFCT_PINS_AS_GPIOS=y` required for DRV2605 (uses NFC pins)
- Logging disabled in production (`CONFIG_LOG=n`) for performance

## Calibration & Storage System

### Interactive Calibration Mode
Hold **STICK_CLICK + PAD_CLICK for 3 seconds** to enter calibration:
- Phase 0 (2s): Center stick, hold trigger released
- Phase 1 (8s): Move stick in circles, pull trigger fully

Code: [main.c lines 2161-2280](src/main.c#L2161-L2280). Uses `analog_driver_begin_calibration_collection()` → `analog_driver_update_calibration_data()` → `analog_driver_finalize_calibration()`.

**Calibration data persists in NVS flash** via `controller_storage_save_calibration()`. Always load on boot with `controller_storage_load_calibration()`.

### Storage Architecture
Uses Zephyr Settings subsystem (NVS backend) with three namespaces:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tommybee456/OpenSplitDeck](https://github.com/tommybee456/OpenSplitDeck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
