---
trigger: always_on
description: Firmware for ESP32-C3 / ESP32-S3 that controls a 2-axis animatronic tail with motion patterns, LED effects, IMU sensing, and BLE configuration.
---

# TailFirmware

Firmware for ESP32-C3 / ESP32-S3 that controls a 2-axis animatronic tail with motion patterns, LED effects, IMU sensing, and BLE configuration.

## Build

Requires: ESP-IDF v5.4.1 (installed via Espressif Windows installer at `C:\Espressif`).
Both the RISC-V (C3) and Xtensa (S3) toolchains must be installed; `build.bat`
puts both on PATH.

```bash
# From ESP-IDF CMD prompt (esp32s3 is equally valid everywhere esp32c3 appears):
idf.py set-target esp32c3
idf.py build
idf.py flash monitor

# From Git Bash (via wrapper):
cmd.exe //c "build.bat set-target esp32c3"
cmd.exe //c "build.bat build"
cmd.exe //c "build.bat flash monitor"
```

Output: `build/TailFirmware.bin` — roughly 0.7-0.8 MB, but the exact size depends
on the target and the build, so don't quote a number: `idf.py size` prints it.

### Dual-target layout

`set-target` selects everything chip-specific; there is nothing else to edit.
All application code is shared — only these three places branch, and a change to
one usually needs the matching change in the others:

| | ESP32-C3 | ESP32-S3 |
|---|---|---|
| Pin map (`main/config/pin_config.h`) | SDA/SCL 7/8, TMC TX/RX 3/9, EN 5, DIAG 6, strip 4, status 10 | SDA/SCL 8/9, TMC TX/RX 17/18, EN 16, DIAG 15, strip 38, status 21 |
| `sdkconfig.defaults.<target>` | 2 MB flash, `partitions.csv` | 4 MB flash, `partitions_4mb.csv` |
| App slots | 960 KB ×2 | 1984 KB ×2 |
| Cores | 1 (RISC-V) | 2 (Xtensa) — task stacks scaled ×1.5 in `main.c` |

`pin_config.h` `#error`s on any other target rather than silently inheriting a
map. Both targets are built in CI (`firmware-build` matrix).

The cross-task hand-offs (`CommandQueue`, `MotionBus`, `FftBuffer`) are
single-producer/single-consumer with release/acquire ordering and are safe on
two cores, which the host stress test exercises on genuinely parallel threads
under ThreadSanitizer. The shared *subsystem state* is not: `motion`, `led_rend`
and `config` swap and resize `MotionPattern`/`LedEffect` instances and
`LedMatrix::coords_` with no lock, relying on single-core priority preemption,
so all three are pinned to core 0 on both targets (`main.c`). Unpinning them
needs retirement slots for those three objects first.

## Test

Host-based unit + integration tests (no hardware/ESP-IDF needed) live in `test/`.
They compile the real C++ app logic against fakes for the ESP-IDF hardware layer.

```bash
./test/run_tests.sh        # configure + build + ctest (Git Bash / Linux / macOS)
# or manually:
cmake -S test/host -B build/host-tests -G Ninja
cmake --build build/host-tests
ctest --test-dir build/host-tests --output-on-failure
```

Requires a C++20 host compiler (e.g. MSYS2 `g++`), CMake, CTest. Unit tests are in
`test/host/unit/`, feature/BLE-path integration tests in `test/host/integration/`,
ESP-IDF fakes in `test/host/fakes/`. See `test/README.md`. Companion-app protocol
improvement backlog: `docs/companion-app-improvements.md`.

## Project Structure

```
main/
  main.c                    Entry point, FreeRTOS task creation, NVS init
  app_bridge.cpp            C/C++ bridge - owns global subsystem instances, builds BLE read payloads
  stepper.c/h               TMC2209 UART driver for 4 stepper motors (VACTUAL velocity + StallGuard/DRV_STATUS read-back, shared-EN freewheel)
  ble_service.c/h           NimBLE GATT server: 14 FF00 characteristics + Battery (0x180F) + Device Info (0x180A)
  drivers/
    i2c_mux.c/h             TCA9548A I2C multiplexer
    encoder_driver.c/h      AS5600 magnetic rotary encoder (drift correction + rehome only; see motion/encoder_assist)
    imu_driver.c/h          BMI270 IMU (accel, gyro, tap detection)
    imu_tap.h               Tap-detector tuning shared with the FF01/FF06 wire contract
    led_strip_driver.c/h    WS2812B via RMT peripheral
    battery_adc.c/h         Pack voltage through a divider on ADC1 (unpopulated -> "not present", never 0 %)
  motion/
    motion_profile.cpp/h    Per-motor jerk-limited profile (max vel/accel/jerk), dead-reckoned position
    axis_controller.cpp/h   2 open-loop motor halves = 1 axis, each driven by a MotionProfile
    axis_mixer.cpp/h        Logical X/Y <-> physical axis targets (MOT-0); default identity
    encoder_assist.cpp/h    AS5600 slow correction + rehome of the dead-reckoned position (MOT-1). NOT a closed loop; off by default
    behavior_engine.cpp/h   State machine above patterns: moods + trigger table (MOT-6). MOTION_PRECEDENCE is defined here
    keyframe_sequence.cpp/h Uploaded timed pose lists, 4 flash slots (MOT-8)
    motion_bus.cpp/h        SPSC snapshot of positions/velocities/gravity/taps for LED effects
    motion_system.cpp/h     2 axes + 2 IMUs + pattern dispatch + StallGuard stall latch (go-limp)
    motion_pattern.h        Virtual base class for patterns
    pid_controller.cpp/h    PID with anti-windup — DEAD CODE. Open-loop motors never used it; protocol v6 removed the PID fields from the config, FF01 (MCMD_SET_PID retired to UNKNOWN_CMD) and FF06. Kept only for its unit test.
    patterns/               11 patterns: static, wagging, loose, idle_sway, excited_wag,
                            circle, figure_eight, shiver, audio_wag, heartbeat, keyframe

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sverredraaisma/TailFirmware](https://github.com/sverredraaisma/TailFirmware) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
