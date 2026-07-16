---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FaderBuddy is a bidirectional motor fader control system with integrated capacitive touch sensing and I2C communication. The hardware consists of a PCB that mounts directly onto Soundwell 60mm motorized faders, with an ATtiny1616 microcontroller managing motor control, touch detection, and I2C communication. Multiple faders can be chained together and controlled by an ESP32 host (designed for ESPHome/Home Assistant integration).

## Project Structure

- **electronics/** - KiCad PCB design files (schematic and board layout)
- **firmware/** - ATtiny1616 firmware (PlatformIO project, Arduino framework)
  - `src/main.cpp` - Main firmware logic with motor control loop and I2C peripheral
  - `src/shared/i2c_data.h` - I2C protocol v5 definitions (shared across all components)
- **esphome/** - ESPHome custom component for Home Assistant integration
  - `components/fader_buddy/` - Core component for interfacing with FaderBuddy boards
  - `examples/multi-fader-display.yaml` - ESP32-S3 example with LVGL display
- **software/** - Software tools and demos
  - `mcp2221-webhid/` - WebHID-based browser demo tool for I2C control via MCP2221 USB-to-I2C bridge
- **production_tools/programAndTest/** - ESP32-based production test fixture
  - ESP32 PlatformIO project with display and current monitoring
  - Python test scripts for automated hardware validation
- **ci/** - Python scripts for electronics export (JLCPCB files, PDFs, renders)
  - Automated workflow for PCB fabrication files, S3 upload

## Build Commands

### Firmware (ATtiny1616)

**IMPORTANT**: All PlatformIO commands require activating the pipenv environment first:
```bash
source ~/.platformio/penv/bin/activate
```

Then you can use PlatformIO commands - make sure to cd to the repo root before running these (there are multiple platformio setups within the repo).
```bash
# Build firmware
pio run

# Upload firmware via UPDI (default port: /dev/ttyUSB0)
pio run --target upload

# Monitor serial output (default port: /dev/ttyUSB1)
pio run --target monitor

# Clean build
pio run --target clean
```

The firmware uses UPDI programming via a USB-to-serial adapter. Upload port and monitor port can be configured in `platformio.ini`.

Generally we don't have the serial RX/TX lines hooked up, so prefer to debug firmware on the ATtiny1616 through other means than serial when possible.


### Firmware (production programAndTest ESP32 jig)

**IMPORTANT**: All PlatformIO commands require activating the pipenv environment first:
```bash
source ~/.platformio/penv/bin/activate
```

You can use PlatformIO commands to build, upload, and view the serial monitor; just cd into the production_tools/programAndTest directory before running these commands.

e.g. `source ~/.platformio/penv/bin/activate && cd production_tools/programAndTest && pio run --target upload`


### Electronics Export

Electronics artifacts (JLCPCB files, schematics, PDFs, 3D renders) are generated via Python scripts in `ci/electronics/`:

```bash
# Install dependencies (KiCad, KiBot, etc) -- only to be used within CI by github runners!
./ci/electronics/dependencies.sh

# Generate JLCPCB fabrication files (gerbers, BOM, CPL)
./ci/electronics/export_jlcpcb.py --release-prefix releases/electronics/ \
  --assembly-schematic electronics/fader_buddy_main.kicad_sch \
  electronics/fader_buddy_main.kicad_pcb

# Generate PCB overview PDF
./ci/electronics/generate_pdf.py --release-prefix releases/electronics/ \
  electronics/fader_buddy_main.kicad_pcb
```

CI automatically exports all electronics artifacts on push (see `.github/workflows/electronics.yml`).

### ESPHome Component

The ESPHome component provides Home Assistant integration with full layer and haptic support:

```bash
# Deploy to ESP32 device
cd esphome
esphome run examples/multi-fader-display.yaml
```

**Key Features:**
- Layer-aware automation triggers: `manual_move`, `touch_change`, `double_tap`
- Per-layer haptic configuration (detent count, strength, mode)
- Per-layer position restore on layer changes
- Rate limiting per sensor (value_change_min_interval)
- Protocol v5 compatible with layer-addressed registers

**Example Configuration:**
See `esphome/examples/multi-fader-display.yaml` for ESP32-S3 with LVGL display integration.

**Important:** Copy `esphome/examples/secrets.yaml.example` to `secrets.yaml` and fill in your WiFi credentials.

### WebHID Demo Tool

Browser-based control interface for testing and diagnostics:

```bash
cd software/mcp2221-webhid
npm install
npm run build
# Open index.html in Chrome/Edge browser
```

**Requirements:**
- Chromium-based browser (Chrome 89+, Edge 89+) with WebHID support
- MCP2221 USB-to-I2C bridge device
- Linux: Install udev rules (`99-mcp2221.rules`) for non-root access

**Features:**
- Real-time position monitoring (10Hz polling)
- Touch sensor diagnostics (raw/delta/reference values)
- Self-calibration trigger
- Layer switching and per-layer position control
- Direct I2C register read/write

## Architecture

### I2C Protocol


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scottbez1/FaderBuddy](https://github.com/scottbez1/FaderBuddy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
