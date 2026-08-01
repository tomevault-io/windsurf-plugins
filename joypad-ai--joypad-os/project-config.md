---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Joypad OS (formerly **USBRetro**) is firmware for RP2040 and ESP32-S3 based adapters that provides universal controller I/O. Old code/commits may reference `USBR_BUTTON_*` or `usbretro` naming.

**Inputs:**
- USB HID controllers, keyboards, mice
- USB X-input (Xbox controllers)
- Bluetooth controllers (via USB BT dongle or Pico W)
- WiFi controllers (via JOCP protocol on Pico W)
- Native controllers (SNES, N64, GameCube via joybus, LodgeNet via PIO)

**Outputs:**
- Retro consoles: PCEngine, GameCube, Dreamcast, Nuon, 3DO, Loopy
- USB Device: HID gamepad, XInput, DirectInput, PS3/PS4/Switch modes
- UART: ESP32 Bluetooth bridge

Uses TinyUSB for USB, BTstack for Bluetooth, LWIP for WiFi networking, and RP2040 PIO for timing-critical console protocols. ESP32-S3 support uses ESP-IDF with FreeRTOS for BLE-to-USB applications.

## Build Commands

### Quick Start

```bash
# One-time setup
brew install --cask gcc-arm-embedded cmake git

# Clone and initialize
git clone https://github.com/joypad-ai/joypad-os.git
cd joypad-os
make init

# Build apps (append _kb2040, _rp2040zero, etc. for specific boards)
make usb2pce_kb2040    # USB/BT → PCEngine
make usb2gc_kb2040     # USB/BT → GameCube
make usb2dc_kb2040     # USB/BT → Dreamcast
make usb2nuon_kb2040   # USB/BT → Nuon
make usb23do_rp2040zero # USB/BT → 3DO
make usb2usb_feather   # USB/BT → USB HID
make snes2usb_kb2040   # SNES → USB HID
make n642usb_kb2040    # N64 → USB HID
make gc2usb_kb2040     # GameCube → USB HID
make lodgenet2usb_pico # LodgeNet → USB HID (Pico)
make lodgenet2usb_pico2 # LodgeNet → USB HID (Pico 2)
make n642dc_kb2040     # N64 → Dreamcast
make bt2usb_pico_w     # BT-only → USB HID (Pico W)
make bt2usb_xiao_esp32s3    # BLE-only → USB HID (ESP32-S3, requires ESP-IDF)
make bt2usb_seeed_xiao_nrf52840   # BLE-only → USB HID (Seeed XIAO nRF52840, requires NCS)
make mouthpad_aprbrother_nrf52840 # Augmental MouthPad BLE → USB HID + NUS relay (April Brother dongle, NCS)
make mouthpad_pico_w              # Augmental MouthPad BLE → USB HID + NUS relay (Pico W)
make wifi2usb_pico_w   # WiFi → USB HID (Pico W)
make controller_btusb_pico_w        # GPIO+JoyWing → BLE+USB HID (Pico W)
make controller_btusb_rp2040_abb    # GPIO+USB Host → USB HID (ABB Passthrough)

# Build all (RP2040 targets only)
make all
make clean

# Flash (macOS - looks for /Volumes/RPI-RP2)
make flash              # Flash most recent build
make flash-usb2pce_kb2040  # Flash specific app

# ESP32-S3 (requires ESP-IDF, see .dev/docs/esp32-port.md)
make flash-bt2usb_xiao_esp32s3  # Flash via esptool
```

Output: `releases/joypad_<commit>_<app>_<board>.uf2`

### App Build Matrix

# Flash (macOS - looks for /Volumes/RPI-RP2)
make flash              # Flash most recent build
make flash-usb2pce_kb2040  # Flash specific app

# ESP32-S3 (requires ESP-IDF, see .dev/docs/esp32-port.md)
make flash-bt2usb_xiao_esp32s3  # Flash via esptool
```

Output: `releases/joypad_<commit>_<app>_<board>.uf2`

### App Build Matrix

| App | Board | Input | Output |
|-----|-------|-------|--------|
| `usb2pce` | KB2040 | USB/BT | PCEngine |
| `usb2gc` | KB2040 | USB/BT | GameCube |
| `usb2dc` | KB2040 | USB/BT | Dreamcast |
| `usb2nuon` | KB2040 | USB/BT | Nuon |
| `usb23do` | RP2040-Zero | USB/BT | 3DO |
| `usb2loopy` | KB2040 | USB/BT | Loopy |
| `usb2usb` | Feather/RP2040-Zero | USB/BT | USB HID |
| `bt2usb` | Pico W/Pico 2 W/ESP32-S3/XIAO nRF52840 | BT/BLE | USB HID |
| `mouthpad` | April Brother nRF52840/Pico W/Pico 2 W | BLE (Augmental MouthPad) | USB HID (SInput) + NUS relay (CDC) |
| `wifi2usb` | Pico W/Pico 2 W | WiFi (JOCP) | USB HID |
| `snes2usb` | KB2040 | SNES | USB HID |
| `n642usb` | KB2040 | N64 | USB HID |
| `gc2usb` | KB2040/RP2040-Zero/Pico | GameCube | USB HID |
| `lodgenet2usb` | Pico/Pico 2 | LodgeNet (N64/GC/SNES) | USB HID |
| `n642dc` | KB2040 | N64 | Dreamcast |
| `snes23do` | RP2040-Zero | SNES | 3DO |
| `usb2uart` | KB2040 | USB | UART/ESP32 |
| `controller_*` | Various | GPIO | USB HID |
| `controller_btusb` | Pico W/Pico 2 W/ESP32-S3/nRF52840 | GPIO | BLE + USB HID |
| `controller_btusb_rp2040_abb` | RP2040 ABB (Passthrough) | GPIO + USB Host | USB HID |

## Architecture

### Repository Structure

```
src/
├── main.c                      # RP2040 entry point, main loop
├── CMakeLists.txt              # RP2040 build configuration
├── platform/
│   ├── platform.h              # Platform HAL (time, identity, reboot)
│   ├── rp2040/platform_rp2040.c
│   └── esp32/platform_esp32.c
├── core/                       # Shared firmware infrastructure
│   ├── buttons.h               # JP_BUTTON_* definitions (W3C order)
│   ├── input_event.h           # Unified input event structure
│   ├── output_interface.h      # Output abstraction
│   ├── router/                 # Input→Output routing
│   │   └── router.c/h          # SIMPLE/MERGE/BROADCAST modes
│   └── services/
│       ├── players/            # Player slot management + feedback
│       ├── profiles/           # Button remapping profiles
│       ├── codes/              # Button sequence detection
│       ├── hotkeys/            # Button combo detection

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joypad-ai/joypad-os](https://github.com/joypad-ai/joypad-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
