---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Nordic Pin Planner is an **unofficial** web-based tool for visualizing and planning pin assignments for Nordic Semiconductor's nRF54L series microcontrollers. It generates Zephyr RTOS board definition files based on user configurations.

**Important**: This is NOT an official Nordic Semiconductor application. All configurations must be verified against official documentation.

## Development Commands

### Install dependencies

```bash
npm install
```

### Formatting

```bash
npm run format          # Auto-fix formatting
npm run format:check    # Check formatting (CI)
```

### Validation & Testing

```bash
npm run validate:schemas   # Validate MCU JSON against mcuSchema.json
npm run smoke-test         # Smoke test MCU data integrity
npm test                   # Run all checks (format + schema + smoke)
```

### Devkit Extraction (requires local Zephyr checkout)

```bash
npm run extract-devkits -- --zephyr-path=/path/to/zephyr
```

### Running the Application

This is a static web application. Open `index.html` in a web browser or use a local development server:

```bash
python -m http.server 8000
# or
npx http-server
```

## Architecture

### Module Structure (`js/`)

The application uses native ES modules (`<script type="module">`). No bundler required.

| Module                   | Purpose                                                       |
| ------------------------ | ------------------------------------------------------------- |
| `js/main.js`             | Entry point: event wiring, theme setup, initialization        |
| `js/state.js`            | Centralized state object, persistence (save/load/reset)       |
| `js/mcu-loader.js`       | MCU/package loading, `initializeApp`, `reinitializeView`      |
| `js/peripherals.js`      | Peripheral organization, toggle, oscillator config, filtering |
| `js/pin-layout.js`       | Responsive pin diagram rendering, pin display updates         |
| `js/devicetree.js`       | All DeviceTree generation functions (30+)                     |
| `js/export.js`           | Board info modal, ZIP assembly, overlay export                |
| `js/console-config.js`   | Serial console UART selection and warnings                    |
| `js/devkit-loader.js`    | Load devkit configs, overlay generation mode                  |
| `js/utils.js`            | Shared utilities (scroll wheel, `parsePinName`)               |
| `js/ui/modals.js`        | Pin selection modal, GPIO modal                               |
| `js/ui/selected-list.js` | Selected peripherals list rendering                           |
| `js/ui/import-export.js` | JSON config import/export modals                              |
| `js/ui/notifications.js` | Toast notification system                                     |

### Other Key Files

- **index.html**: Main application UI with modals
- **style.css**: Complete styling including dark mode, responsive breakpoints, toast styles
- **mcus/**: MCU package definitions and devicetree templates
- **devkits/**: Pre-extracted devkit pin configurations (JSON)
- **ci/**: CI validation scripts (schema validation, smoke tests, board generation)
- **.github/workflows/**: GitHub Actions CI/CD pipelines

### MCU Data Architecture

The application uses a hierarchical JSON-based system:

1. **manifest.json**: Top-level MCU catalog
   - Lists all supported MCUs (nRF54L05, nRF54L10, nRF54L15, nRF54LV10A, nRF54LM20A)
   - Maps MCUs to available packages
   - Defines which MCUs support non-secure builds (`supportsNonSecure`)
   - Defines which MCUs support FLPR (Fast Lightweight Processor) core (`supportsFLPR`)

2. **Package Definition Files** (e.g., `qfn48-6x6-qfaa.json`):
   - Physical chip dimensions and rendering parameters
   - Pin layout strategy (quadPerimeter with pin counts per side)
   - Pin definitions with GPIO mappings
   - Peripheral configurations with signal-to-pin mappings
   - Address space information for memory-mapped peripherals

3. **devicetree-templates.json**: Per-MCU Zephyr DeviceTree generation templates
   - Maps peripheral IDs to DeviceTree node names
   - Defines signal-to-pinctrl mappings
   - Provides templates for generating `.dtsi` files

### State Management

All state is centralized in `js/state.js` via a single exported `state` object:

- `state.mcuManifest`: Loaded from manifest.json at startup
- `state.mcuData`: Currently selected MCU package data
- `state.selectedPeripherals`: Array of user-selected peripherals with pin assignments
- `state.usedPins`: Tracks which pins are assigned to prevent conflicts
- `state.usedAddresses`: Tracks address space usage for peripherals
- `state.deviceTreeTemplates`: Loaded per-MCU for export generation
- `state.consoleUart`: Selected console UART peripheral ID (null = RTT mode)
- `state.devkitConfig`: Loaded devkit configuration (null = custom board mode)

### Key Application Flows

#### 1. Initialization (initializeApp in mcu-loader.js)

- Fetches `mcus/manifest.json`
- Populates MCU selector dropdown
- Triggers initial MCU/package load

#### 2. MCU/Package Selection (mcu-loader.js)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hlord2000/PinPlanner](https://github.com/hlord2000/PinPlanner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
