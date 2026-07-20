---
trigger: always_on
description: This repository contains JavaScript examples for Shelly smart home devices. Scripts run on Shelly Gen1/Gen2/Gen3 devices using their embedded scripting engine (mJS - restricted JavaScript).
---

# Shelly Script Examples - Agent Guidelines

## Overview

This repository contains JavaScript examples for Shelly smart home devices. Scripts run on Shelly Gen1/Gen2/Gen3 devices using their embedded scripting engine (mJS - restricted JavaScript).

## Project Structure

```
shelly-script-examples/
|-- ble/                        # BLE/BLU sensor and button examples
|-- lora/                       # LoRa communication and device control
|-- mqtt/                       # MQTT and Home Assistant integrations
|-- power-energy/               # Power monitoring and load management
|-- switch-input/               # Input, switch, and cover control scripts
|-- weather-env/                # Weather and environmental integrations
|-- http-integrations/          # HTTP endpoints and external services
|-- networking/                 # Provisioning and watchdog scripts
|-- scheduling/                 # Scheduling, scenes, and orchestration
|-- blu-assistant/              # Shelly BLU Assistant device management
|-- examples-manifest.json      # Central registry of all examples (IMPORTANT)
|-- howto/                      # Basic tutorials and minimal examples
|-- snippets/                   # Reusable code snippets (JSON format)
|-- tools/                      # Upload utilities (Python/Bash)
`-- .github/                    # CI/CD workflows and issue templates
```

## Script Categories

| Category | Examples | Description |
|----------|----------|-------------|
| **BLE/Bluetooth** | aranet4, ruuvi, bparasite, shelly-blu-* | BTHome protocol, sensor reading |
| **MQTT** | mqtt-discovery, mqtt-announce | Home Assistant integration |
| **Home Automation** | hue-lights, load-shedding | Scene control, power management |
| **LoRa** | lora/* | Long-range communication |
| **Utilities** | power-*, scheduler-*, weather-* | Monitoring, scheduling |
| **Blu Assistant** | blu-assistant/*.shelly.js | Virtual component management |

---

## Coding Standards

### Single File Application
- **Each script is standalone**: Every `.shelly.js` file is a complete, self-contained application
- **No imports or includes**: Shelly mJS does not support importing code from other files
- **No shared dependencies**: Each script must contain all the code it needs

### File Naming
- Use descriptive kebab-case names: `ble-shelly-motion.shelly.js`, `mqtt-discovery.shelly.js`
- Reflect the script's purpose in the filename

### Code Style (Enforced via .editorconfig/.prettierrc)
- **2-space indentation** (not tabs)
- **Single quotes** for strings
- **Semicolons required**
- **LF line endings**
- **UTF-8 charset**

### Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Variables | camelCase | `lastTime`, `switchStatus` |
| Constants | UPPER_SNAKE_CASE | `CONFIG`, `DEVICE_ID` |
| Functions | camelCase | `turnOff`, `handleEvent` |
| Event handlers | prefix with `on` | `onButtonPress`, `onStatusChange` |
| Boolean functions | prefix with `is`/`has` | `isValidMac`, `hasPermission` |

### Code Structure Pattern

Scripts follow a **two-header pattern** for documentation:

1. **Standard Header** - Machine-readable metadata for manifest/index generation
2. **Detailed Documentation** - Human-readable technical details (hardware, protocol, etc.)

```javascript
/**
 * @title Human-Readable Title
 * @description Brief description of what the script does. Keep it concise
 *   (1-2 sentences). Mention firmware requirements if applicable.
 * @status production
 * @link https://github.com/ALLTERCO/shelly-script-examples/blob/main/path/to/file.shelly.js
 */

/**
 * Detailed Documentation Block
 *
 * Extended explanation of the script's purpose and functionality.
 *
 * Hardware Connection: (for hardware-interfacing scripts)
 * - Device TX -> Shelly RX (GPIO)
 * - Device RX -> Shelly TX (GPIO)
 * - VCC -> 3.3V or 5V
 * - GND -> GND
 *
 * Protocol: (for communication scripts)
 * - Baud rate: 9600
 * - Frame format: [Header] [Length] [Data] [Checksum]
 *
 * Components Created: (for setup scripts)
 * - Text:200    - Status display
 * - Number:200  - Value display
 * - Button:200  - Control button
 *
 * @see https://link-to-reference-documentation
 */

// ============================================================================
// CONFIGURATION
// ============================================================================

let CONFIG = {
  // User-configurable options at top
  deviceId: 0,
  timeout: 5000,
};

// ============================================================================
// STATE
// ============================================================================

let state = {
  lastUpdate: null,
  isRunning: false,
};

// ============================================================================
// HELPER FUNCTIONS
// ============================================================================

function helperFunction(param) {
  // Implementation
}

// ============================================================================
// MAIN LOGIC
// ============================================================================

function main() {
  // Core logic
}


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ALLTERCO/shelly-script-examples](https://github.com/ALLTERCO/shelly-script-examples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
