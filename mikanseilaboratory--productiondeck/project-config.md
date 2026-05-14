---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with the ProductionDeck repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with the ProductionDeck repository.

## Project Overview

ProductionDeck is an open-source RP2040-based Stream Deck–compatible firmware in Rust (Embassy). Multiple **device profiles** (Mini, Classic/Mk.2, XL, Neo, +, modules, etc.) share the same codebase; each profile selects USB PID, layout, and **Legacy (Mini)** vs **Main/Expanded** HID handling. Physical hardware is often a small key matrix + one ST7735 region unless you build a larger layout.

**Current Status**: Alpha - Firmware compiles successfully, ready for hardware testing.

## Quick Start Commands

### Essential Build Commands
```bash
# Check compilation without building
cargo check

# Build in debug mode
cargo build

# Build in release mode (recommended for embedded)
cargo build --release

# Format code
cargo fmt

# Run linter
cargo clippy
```

### Development Commands
```bash
# Clean build artifacts
cargo clean

# Update dependencies (use with caution)
cargo update

# Generate documentation
cargo doc --open
```

## Prerequisites

1. **Rust Toolchain** - Latest stable Rust (1.75+ recommended)
2. **thumbv6m-none-eabi target** - `rustup target add thumbv6m-none-eabi`
3. **elf2uf2-rs** - `cargo install elf2uf2-rs` (for UF2 conversion)
4. **flip-link** - `cargo install flip-link` (stack overflow protection)

## Project Structure

### Core Source Files
- `src/bin/*.rs` - One binary per target device (`mini`, `xl`, `mk2`, `neo`, `plus-xl`, …)
- `src/lib.rs` - Library root (`productiondeck` crate)
- `src/config.rs` - Hardware configuration constants and pin assignments
- `src/device/mod.rs` - USB PID, layout, and protocol family per `Device`
- `src/protocol/v1.rs` / `v2.rs` / `module_6.rs` - HID protocol handlers
- `src/usb.rs` - USB HID class and routing to protocol + channels
- `src/display.rs` - Display handling and graphics rendering
- `src/buttons.rs` - Button matrix / direct scanning

### Configuration Files
- `Cargo.toml` - Rust project manifest and dependencies
- `.cargo/config.toml` - Build configuration and target settings
- `memory.x` - RP2040 memory layout for linker
- `build.rs` - Build script for memory layout

### Documentation
- `README.md` - Main project documentation
- `LICENSE` - MIT License
- `StreamDeck_Protocol_Reference.md` - Protocol documentation (Elgato HID API alignment)

## Architecture Overview

### Hardware Configuration
- **Target**: Raspberry Pi Pico (RP2040 dual-core ARM Cortex-M0+)
- **USB Identity**: VID `0x0FD9` (Elgato); PID depends on selected `Device` (see `src/device/mod.rs`)
- **Display**: Often 1× ST7735; layout/size depend on profile (`display_config`)
- **Buttons**: Matrix or direct wiring per `hardware.rs` / `buttons.rs`
- **Protocol**: Legacy Mini family (`v1`) or Main/Expanded family (`v2`) per Elgato HID docs

### Core Architecture
- **Async Embassy framework**: Modern Rust async/await for embedded
- **Dual-core design**: Core 0 handles USB/protocol, Core 1 handles displays/buttons
- **USB HID interface**: Stream Deck Mini (legacy) or Main protocol (JPEG + feature reports) per build
- **Channel communication**: Embassy channels for inter-task communication
- **Hardware abstraction**: Configurable pin assignments via `config.rs`

### Pin Assignments (Critical for Hardware)
```
RP2040 Pin Layout:
├── Buttons (Matrix Scan):
│   ├── ROW0: GP2  ┐
│   ├── ROW1: GP3  ├─ Button Matrix (3x2)
│   ├── COL0: GP4  │
│   ├── COL1: GP5  │
│   └── COL2: GP6  ┘
│
├── SPI Display (ST7735):
│   ├── MOSI: GP19 (Data)
│   ├── SCK:  GP18 (Clock)
│   ├── DC:   GP14 (Data/Command)
│   ├── RST:  GP15 (Reset)
│   ├── CS:   GP8  (Chip Select)
│   └── BLK:  GP17 (Backlight PWM)
│
├── Control:
│   ├── Status LED: GP25 (Built-in)
│   ├── USB LED:    GP20 (Connection status)
│   └── Error LED:  GP21 (Error indication)
│
└── Debug:
    ├── UART TX: GP0 (Debug output)
    └── UART RX: GP1 (Debug input)
```

## USB Protocol Implementation

The device implements StreamDeck Mini's exact USB HID protocol:
- **Input reports**: Button states (6 bytes for 6 keys)
- **Output reports**: Image data packets (1024 bytes)
- **Feature reports**: Commands (brightness, reset, version)
- **Embassy USB stack** with usbd-hid for HID functionality

### Key Protocol Details
- **VID**: 0x0fd9 (Elgato Systems)
- **PID**: 0x0063 (StreamDeck Mini)
- **Class**: HID (Human Interface Device)
- **Image format**: 80x80 pixels, RGB565 color format

## Dependencies

### Core Framework
- `embassy-rp` - RP2040 hardware abstraction
- `embassy-usb` - USB stack implementation
- `embassy-time` - Time and timer functionality
- `embassy-executor` - Async executor
- `embassy-sync` - Synchronization primitives

### Hardware Abstraction
- `embedded-hal` / `embedded-hal-async` - Hardware abstraction layer
- `embedded-hal-bus` - Bus abstraction for SPI

### Display and Graphics
- `st7735-lcd` - ST7735 display driver
- `embedded-graphics` - Graphics primitives

### USB HID
- `usbd-hid` - USB HID device implementation

### Utilities
- `heapless` - No-alloc data structures

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MikanseiLaboratory/productiondeck](https://github.com/MikanseiLaboratory/productiondeck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
