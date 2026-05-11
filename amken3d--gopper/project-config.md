---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---


# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Gopper is a Klipper firmware implementation written in TinyGo for modern microcontrollers. It aims to provide full compatibility with the Klipper protocol while bringing Go's type safety and modern development practices to embedded 3D printer firmware.

**Status**: Early development, not production-ready.

## Current Status (2025-11)

### Working Features
- ✅ USB CDC communication on RP2040 (Raspberry Pi Pico)
- ✅ Klipper protocol handshake and identification
- ✅ Dictionary compression using custom TinyGo-compatible zlib
- ✅ Command registration and dispatch system
- ✅ VLQ encoding/decoding
- ✅ Basic command handlers (identify, get_uptime, get_clock, get_config)
- ✅ ADC (Analog-to-Digital Converter) support
- ✅ Stepper motor control (config_stepper, queue_step, set_next_step_dir, reset_step_clock)
- ✅ Stepper position reporting (stepper_get_position, stepper_position response)
- ✅ Trigger synchronization (trsync) for coordinated homing
- ✅ Endstop support for homing operations
- ✅ GPIO digital input/output
- ✅ PWM support

### Known Issues
- ⚠️ Stepper PIO backend not fully tested (GPIO fallback works)
- ⚠️ Timer system not fully tested under load
- ⚠️ Actual motor movement not yet verified (protocol communication confirmed working)

### Recent Bug Fixes (2025-11-12)
- **Fixed circular buffer wrap-around bug** causing timeouts after ~6 seconds of communication
  - `FifoBuffer.Data()` was only returning first segment when wrapped, causing message corruption
  - Added proper contiguous data copy for wrapped buffers
- **Fixed firmware restart hanging/crashing issue**
  - `FIRMWARE_RESTART` command was hanging because MCU reset occurred before ACK was sent
  - Root cause: `machine.CPUReset()` (ARM SYSRESETREQ) doesn't properly reset RP2040
  - **Solution**: Use RP2040 watchdog timer for reset instead
    - Watchdog reset properly resets all hardware including USB controller
    - Must call `machine.Watchdog.Start()` after configuring timeout
    - Must disable watchdog on boot to clear previous state
  - Deferred reset until after protocol ACK is transmitted to host
  - Added `resetPending` flag and `CheckPendingReset()` function
  - Klipper now properly receives ACK and reconnects after firmware restart
- **Implemented proper firmware restart with hardware reset**
  - Added `reset` command that triggers watchdog reset for complete MCU reset
  - Uses `machine.Watchdog` API with 1ms timeout for reliable RP2040 reset
  - Watchdog disabled on boot (`TimeoutMillis: 0`) to prevent issues after reset
  - Matches behavior of traditional Klipper firmwares (AVR, STM32)
  - Ensures all hardware peripherals are properly reset, not just software state
- Fixed TinyGo GC issue with compressed dictionary buffer (`bytes.Buffer` data being reclaimed)
- Fixed deadlock in dictionary `GetChunk()` function (RWMutex reentrancy issue)
- Added defensive copying to prevent memory corruption during USB transmission
- Improved USB write failure detection and buffer management
- ADC support now uses `machine.ADC` API instead of custom implementation

## Build Commands

```bash
# Build for RP2040 (Raspberry Pi Pico) - default target
make rp2040

# Build for STM32F4
make stm32f4

# Run tests (protocol and core packages)
make test

# Run tests for specific package
go test ./protocol/...
go test ./core/...

# Clean build artifacts
make clean
```

## Testing Commands

### Unit Tests

```bash
# Run all tests with verbose output
go test -v ./...

# Run tests for a single package
go test -v ./protocol

# Run a specific test
go test -v ./protocol -run TestDecodeVLQ
```

### Hardware Testing (RP2040)

```bash
# Flash firmware to RP2040
# 1. Hold BOOTSEL button and plug in USB
# 2. Copy firmware to mounted drive:
cp build/gopper-rp2040.uf2 /media/[user]/RPI-RP2/

# Test with Klipper console (requires Klipper installed)
~/klippy-env/bin/python ~/klipper/klippy/console.py -v /dev/ttyACM0

# Expected LED flash patterns on boot:
# - 5 flashes: Starting firmware
# - 3 flashes: JSON dictionary generation complete
# - 2 flashes: Compression successful
# - 4 flashes: USB reconnection detected (after disconnect)

# Monitor serial directly (for debugging)
screen /dev/ttyACM0 250000
# or
picocom /dev/ttyACM0 -b 250000
```

## Requirements

- **TinyGo**: 0.31.0 or later (for embedded builds)
- **Go**: 1.21 or later (for tests and development)

## Architecture Overview

### Core System Design

Gopper uses a **command-scheduler-timer** architecture that mirrors Klipper's design for real-time operation:

1. **Protocol Layer** (`protocol/`): Handles Klipper wire protocol communication
   - Variable Length Quantity (VLQ) encoding/decoding for efficient parameter passing
   - Message blocks with sequence numbers and CRC checking
   - Compatible with Klipper's communication protocol

2. **Command System** (`core/command.go`): Command registration and dispatch
   - Commands are registered similar to Klipper's `DECL_COMMAND` macro
   - Each command has an ID, name, format string, and handler function
   - CommandRegistry maintains a dictionary sent to the host for command mapping


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amken3d/gopper](https://github.com/amken3d/gopper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
