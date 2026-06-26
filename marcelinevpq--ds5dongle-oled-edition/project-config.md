---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Raspberry Pi Pico 2 W (RP2350) firmware that bridges a Sony DualSense (DS5) controller over Bluetooth Classic to a USB host. Presents itself to the host as a USB HID + UAC1 audio composite device. This repo is `MarcelineVPQ/DS5Dongle-OLED-Edition`, a personal fork of upstream `awalol/DS5Dongle` that adds an optional Waveshare Pico-OLED-1.3 status display (10 screens) and a 4-slot persistent multi-controller pairing layer.

`README.md` is authoritative for user-facing capability; `CHANGELOG.md` tracks what shipped per release. Read both before proposing changes — much of the design is non-obvious from the code alone (audit history, BT pairing-posture rationale, overclock requirement).

## Build

This is firmware — there are no tests. Verification is hardware-in-the-loop (see "Hardware-in-the-loop workflow" below).

The build is **non-trivial to set up** because it requires a specific TinyUSB version:

```bash
# Pico SDK 2.2.0 must be installed; export PICO_SDK_PATH.
# CRITICAL: TinyUSB must be pinned to 0.20.0. The 0.18.0 that ships with
# Pico SDK 2.2.0 lacks the 4-arg form of TUD_AUDIO_EP_SIZE used by
# src/tusb_config.h — compile errors with no clear pointer to the cause.
cd "$PICO_SDK_PATH/lib/tinyusb"
git fetch --tags && git checkout 0.20.0

# Build
cd /path/to/DS5Dongle
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release -DPICO_SDK_PATH="$PICO_SDK_PATH"
cmake --build build --target ds5-bridge
# Produces build/ds5-bridge-oled.uf2  (output name set in CMakeLists.txt, not the default)
```

Common build flags (set via `-D` on the cmake invocation):

- `ENABLE_SERIAL=ON` — route printf to USB CDC for debugging. Default OFF (production).
- `ENABLE_VERBOSE=ON` — chatty BT/HID logging. Default OFF.
- `ENABLE_BATT_LED=OFF` — disable the low-battery LED blink. Default ON.
- `PICO_W_BUILD=ON` — build for the original Pico W (drops audio, lower clock). Default targets Pico 2 W.

CI runs four build variants in `.github/workflows/build.yml` (standard / debug / no-batt-led / Pico W). When changing build flags or CMake, update the workflow accordingly.

## Hardware-in-the-loop workflow

There are no software tests. To verify a change:

1. Build → produces `build/ds5-bridge-oled.uf2`.
2. Put Pico into bootloader: hold BOOTSEL, plug USB. It mounts as `/run/media/$USER/RP2350` (Linux) or `RPI-RP2` (others).
3. Copy the UF2 to the mount: `cp build/ds5-bridge-oled.uf2 /run/media/$USER/RP2350/` — Pico auto-reboots into the new firmware.
4. Pair a DualSense: hold Create + PS on the controller until the lightbar pulses; the dongle inquiry picks it up.
5. Verify enumeration host-side: `lsusb | grep 054c:0ce6` should show `Sony Corp. DualSense wireless controller (PS5)`.

The development cadence is **one feature per UF2 + checkpoint with the user before moving on** — the user has hardware in front of them, regressions are caught the same minute, and large multi-feature commits make root-causing hard.

## Architecture

**Two cores, asymmetric workload:**

- **Core 0** runs everything: `main()` infinite loop in `src/main.cpp` drives `cyw43_arch_poll()` → BTstack, `tud_task()` → TinyUSB, `audio_loop()` (USB → BT haptic path), `interrupt_loop()` (BT → USB HID path), and `oled_loop()`.
- **Core 1** runs only `core1_entry()` in `src/audio.cpp` — the Opus speaker encoder. The two cores communicate via Pico SDK `queue_t` (`audio_fifo`) and a critical section (`opus_cs`).

**Two parallel data paths through the firmware:**

1. **HID** (DS5 input → host, host output → DS5): `src/bt.cpp` registers `on_bt_data()` → `src/main.cpp:on_bt_data()` → updates `interrupt_in_data[63]` → `tud_hid_report()`. Output reports come the other way through `tud_hid_set_report_cb`.
2. **Audio** (host USB audio → DS5 speaker + haptics): `src/audio.cpp:audio_loop()` reads 4-channel USB UAC1, splits ch1/2 (speaker → Opus encode on core1 → BT 0x32 report) and ch3/4 (haptic → resample 48k→3k → BT 0x32 report).

**BT layer:** Uses Pico SDK's bundled BTstack (no separate clone). `src/bt.cpp` is the only file that touches HCI/L2CAP. PSMs are `PSM_HID_CONTROL` and `PSM_HID_INTERRUPT`. Link keys live in BTstack's TLV NVM with `NVM_NUM_LINK_KEYS=4` (configured in `src/btstack_config.h`).

**OLED add-on is optional and self-contained:** All 10 screens, the SH1107 SPI driver, the 5×7 font, the icon table, and the input handling live in `src/oled.cpp` (~30 KB). The only outward dependencies are read-only accessors (`bt_is_connected`, `bt_get_addr`, `audio_peak_*`, `bt_get_signal_strength`, the new slot accessors) and the global `interrupt_in_data[63]` (read-only for input visualization). When no OLED is wired, the SPI writes go nowhere — no init check needed.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MarcelineVPQ/DS5Dongle-OLED-Edition](https://github.com/MarcelineVPQ/DS5Dongle-OLED-Edition) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
