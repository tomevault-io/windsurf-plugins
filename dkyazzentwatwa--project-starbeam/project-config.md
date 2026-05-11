---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Quick Reference Commands

```bash
# Compile and upload V2 firmware (ALWAYS use huge_app partition!)
arduino-cli compile --upload --fqbn esp32:esp32:esp32:PartitionScheme=huge_app -p /dev/cu.SLAB_USBtoUART starbeam_v2/

# Test compilation without hardware
./test_build.sh

# Monitor serial output
screen /dev/cu.SLAB_USBtoUART 115200

# Port names: macOS=/dev/cu.*, Linux=/dev/ttyUSB0, Windows=COM3
```

## Project Overview

Project Starbeam V2 is a multi-band RF signal intelligence platform built around the ESP32-WROOM-32D microcontroller with multiple RF modules (NRF24L01 and CC1101) for signal analysis, generation, and manipulation across 2.4 GHz and 433 MHz bands. It includes optional HackRF One integration for extended frequency coverage up to 6 GHz.

**CRITICAL LEGAL NOTICE**: This is a security research and educational platform. RF transmission, jamming operations, and WiFi security testing features are strictly regulated. Only operate on permitted frequencies with proper authorization. This tool is intended for authorized security testing, educational contexts, and research environments.

## Important: Repository Structure

This repository contains **two firmware versions**:

- **starbeam_v1/** - Original monolithic firmware (~1000+ line single .ino file)
- **starbeam_v2/** - Modular refactored firmware (current development focus)

Unless explicitly working on V1, **all development should target starbeam_v2/**.

## Development Environment Setup

### Arduino IDE Configuration

1. **Install Arduino IDE 2.x** from https://www.arduino.cc/en/software/
2. **Add ESP32 Board Support**:
   - File → Preferences → Additional Board Manager URLs
   - Add: `https://dl.espressif.com/dl/package_esp32_index.json`
   - Tools → Board → Boards Manager → Install "esp32" by Espressif Systems

3. **Required Libraries** (Install via Arduino Library Manager):
   - `Adafruit GFX Library`
   - `Adafruit SSD1306`
   - `U8g2_for_Adafruit_GFX`
   - `RF24` by TMRh20
   - `ELECHOUSE_CC1101_SRC_DRV` by LSatan
   - `ESP32 BLE Arduino` (included with ESP32 board support)

4. **Manual Library Installation**:
   - The `SmartRC-CC1101-Driver-Lib2/` directory is already in the repository
   - Dual CC1101 radio support via `ELECHOUSE_CC1101_SRC_DRV2.h`

### Board Configuration

- **Board**: ESP32 Dev Module
- **Upload Speed**: 921600
- **CPU Frequency**: 240MHz
- **Flash Frequency**: 80MHz
- **Flash Mode**: QIO
- **Flash Size**: 4MB (32Mb)
- **Partition Scheme**: **Huge APP (3MB No OTA/1MB SPIFFS)** - REQUIRED for security features
- **Core Debug Level**: None (or "Verbose" for debugging)

**IMPORTANT**: The `huge_app` partition scheme is **required** for V2 firmware. It allocates 3MB for program storage instead of the default 1.2MB, which is necessary for WiFi attack features and web server functionality.

### Building and Uploading

#### Arduino CLI (Recommended)

```bash
# Install Arduino CLI (if not already installed)
# macOS: brew install arduino-cli
# Linux: curl -fsSL https://raw.githubusercontent.com/arduino/arduino-cli/master/install.sh | sh
# Windows: Download from https://arduino.github.io/arduino-cli/

# Install ESP32 platform (first time only)
arduino-cli core update-index
arduino-cli core install esp32:esp32

# Install required libraries (first time only)
arduino-cli lib install "Adafruit GFX Library"
arduino-cli lib install "Adafruit SSD1306"
arduino-cli lib install "U8g2_for_Adafruit_GFX"
arduino-cli lib install "RF24"
arduino-cli lib install "ELECHOUSE_CC1101_SRC_DRV"

# Compile only (no upload)
arduino-cli compile --fqbn esp32:esp32:esp32:PartitionScheme=huge_app starbeam_v2/

# Compile and upload in one command
arduino-cli compile --upload --fqbn esp32:esp32:esp32:PartitionScheme=huge_app -p /dev/cu.SLAB_USBtoUART starbeam_v2/

# Or separate compile and upload
arduino-cli compile --fqbn esp32:esp32:esp32:PartitionScheme=huge_app starbeam_v2/
arduino-cli upload --fqbn esp32:esp32:esp32:PartitionScheme=huge_app -p /dev/cu.SLAB_USBtoUART starbeam_v2/

# Port examples:
# macOS:   /dev/cu.SLAB_USBtoUART or /dev/cu.usbserial-*
# Linux:   /dev/ttyUSB0 or /dev/ttyACM0
# Windows: COM3 or COM4 (check Device Manager)
```

#### Arduino IDE

```bash
# Open: starbeam_v2/starbeam_v2.ino
# Tools → Board → ESP32 Arduino → ESP32 Dev Module
# Tools → Partition Scheme → Huge APP (3MB No OTA/1MB SPIFFS)  ← REQUIRED
# Tools → Port → Select your ESP32 port
# Sketch → Upload (Ctrl+U / Cmd+U)
# Hold BOOT button on ESP32 during upload if auto-reset fails
```

#### Automated Build Testing (No Hardware)

```bash
# Run from project root directory
./test_build.sh

# This script will:
# - Check Arduino CLI installation
# - Verify ESP32 platform is installed
# - Auto-install missing libraries
# - Compile firmware with huge_app partition
# - Report memory usage (should be ~1.81 MB / 3 MB)
```

### Serial Monitor

```bash
# Arduino IDE: Tools → Serial Monitor, set to 115200 baud

# External serial monitor:
screen /dev/cu.usbserial-* 115200
# or
minicom -D /dev/cu.usbserial-* -b 115200
```

## Code Architecture (V2)

### Modular Component Structure


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dkyazzentwatwa/project-starbeam](https://github.com/dkyazzentwatwa/project-starbeam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
