---
trigger: always_on
description: This repository is a Homebridge platform plugin that bridges TP-Link Kasa/Tapo smart devices (plugs, switches, bulbs, power strips, light strips) to Apple HomeKit. It uses a hybrid architecture: the main orchestration, accessory management, and Homebridge integration is in TypeScript/Node.js; direct device communication and control is handled by Python scripts using the [python-kasa](https://github.com/python-kasa/python-kasa) library, exposed via a local API.
---

# Homebridge Kasa Python - Copilot Coding Agent Onboarding

## Repository Overview

This repository is a Homebridge platform plugin that bridges TP-Link Kasa/Tapo smart devices (plugs, switches, bulbs, power strips, light strips) to Apple HomeKit. It uses a hybrid architecture: the main orchestration, accessory management, and Homebridge integration is in TypeScript/Node.js; direct device communication and control is handled by Python scripts using the [python-kasa](https://github.com/python-kasa/python-kasa) library, exposed via a local API.

- **Type**: Homebridge plugin for smart home integration
- **Languages**: TypeScript/Node.js (ES Modules), Python 3.11/3.12/3.13
- **Supported Devices**: Plugs, power strips, switches, bulbs, light strips (see README for full list)
- **Target Runtime**: Node.js 20/22/24, Python 3.11/3.12/3.13
- **Size**: ~3,000 lines TypeScript, several Python scripts
- **Main Entry**: `src/index.ts`, `src/platform.ts`
- **Python API Entrypoint**: `src/python/kasaApi.py`, startup: `src/python/startKasaApi.py`
- **Configuration Schema**: `config.schema.json`
- **Linting**: ESLint (config: `eslint.config.mjs`)
- **Build Output**: Compiled JS in `dist/`, Python scripts copied to `dist/python/`

## Environment Setup and Build Instructions

**CRITICAL:** This project **requires both Node.js AND Python environments** set up correctly. All device communication will fail if Python dependencies are missing.

### Environment Setup

1. **Python Setup (ALWAYS FIRST):**
   ```bash
   python3 -m pip install --upgrade pip
   python3 -m pip install -r requirements.txt
   ```
   - If install times out, verify with:
     ```bash
     python3 -c "import kasa; print('Python Import OK')"
     ```

2. **Node.js Setup (Supported: 20/22/24):**
   - Always use **`npm ci`** (never `npm install`)
   ```bash
   npm ci
   ```

### Build, Lint, and Validation (ALWAYS in order):

```bash
npm run lint            # Lint (must pass, zero warnings)
npm run build           # Build (runs: npm ci && rimraf -I ./dist && tsc && node copyPythonFiles.js)
node -e "import('./dist/index.js').then(()=>console.log('Node OK'))"
python3 -c "import kasa; print('Python Import OK')"
```

- **Development/watch**: `npm run watch` (rebuilds and links automatically)
- **Test**: `npm run test` (outputs "No test specified")
- **Lint only**: `npm run lint`

#### Common Troubleshooting

- **Python errors at runtime**: Python dependencies *must* be installed first.
- **rimraf not found**: Run `npm ci` to install dev dependencies.
- **Missing dist/python files**: Build process copies Python files from `src/python/` to `dist/python/`.
- **Pip install timeout**: If network issues, verify using import command above.
- **Build will succeed even if Python deps are missing, but runtime will fail.**

#### Build Timing

- Clean build: ~10-15 seconds; incremental builds are faster.
- Pip install can timeout; verify via import.

## Project Architecture and Layout

**Root files:**
- `package.json` - Node dependencies, scripts
- `tsconfig.json` - TypeScript config  
- `requirements.txt` - Python dependencies
- `eslint.config.mjs` - ESLint config
- `config.schema.json` - Homebridge config schema
- `copyPythonFiles.js` - Copies Python scripts to dist
- `nodemon.json` - Watch config

**Source:**
```
src/
├── index.ts                  # Plugin registration
├── platform.ts               # Main platform logic
├── config.ts                 # Config parsing/validation
├── settings.ts               # Platform/plugin names
├── utils.ts                  # Utility functions
├── taskQueue.ts              # Task management
├── accessoryInformation.ts   # HomeKit accessory info
├── devices/
│   ├── index.ts              # Base accessory class
│   ├── create.ts             # Device creation factory
│   ├── deviceManager.ts      # Device communication manager
│   ├── kasaDevices.ts        # Type definitions
│   ├── homekitPlug.ts        # Plug logic
│   ├── homekitPowerStrip.ts  # Power strip logic
│   ├── homekitSwitch.ts      # Switch logic
│   ├── homekitLightBulb.ts   # Bulb logic
│   └── homekitSwitchWithChildren.ts # Multi-child switches/fans
└── python/
    ├── kasaApi.py            # Python API bridge
    ├── startKasaApi.py       # API server startup
    └── pythonChecker.ts      # Python env validation (Node-side)
```
- **Build output:**  
  - `dist/` - Compiled JS + copied Python files
  - `dist/python/` - Python scripts

## Device Implementation Patterns

- **Device factory:** `src/devices/create.ts` creates HomeKit device instances for discovered Kasa devices.
- **Base class:** `src/devices/index.ts` is the HomeKitDevice base class.
- **Accessory logic:** Device types (plug, bulb, power strip, switch, multi-child) each have dedicated classes.
- **Python bridge:** All device control/state queries use the Python API via HTTP.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZeliardM/homebridge-kasa-python](https://github.com/ZeliardM/homebridge-kasa-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
