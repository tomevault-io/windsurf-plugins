---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

HamLedger is a modern amateur radio logging application built as an Electron desktop app with Vue 3 frontend. It provides QSO (radio contact) logging, real-time rig control via Hamlib, DX cluster integration, WSJT-X integration, and various ham radio utilities.

## Development Commands

### Running the Application
```bash
npm run app:dev          # Start development server with hot reload
npm run app:build        # Build for production
npm run app:preview      # Preview production build
```

### Code Quality
```bash
npm run lint             # Run ESLint
npm run format           # Format code with Prettier
npm run format:check     # Check code formatting
```

### Testing
```bash
npm test                 # Run tests with Vitest
npm run test:coverage    # Run tests with coverage report
```

### Building Distributables
```bash
npm run build:win        # Build Windows installer
npm run build:mac        # Build macOS installer
npm run build:linux      # Build Linux packages (AppImage, deb, rpm)
npm run build:all        # Build for all platforms
```

### TypeScript
```bash
npm run ts               # Compile TypeScript once
npm run watch            # Watch TypeScript files for changes
```

## Architecture Overview

### Electron Architecture

This is a standard Electron app with clear separation between main and renderer processes:

- **Main Process** (`src/electron/main/main.ts`): Handles native operations, IPC handlers, file system access, database operations, and manages background processes (rigctld, WSJT-X UDP server)
- **Preload Script** (`src/electron/preload/preload.ts`): Exposes safe IPC channels to renderer via `window.electronAPI`
- **Renderer Process** (`src/`): Vue 3 application with Pinia state management

### Communication Flow

1. Vue components → Pinia stores → `window.electronAPI` → IPC → Main process
2. Main process background services emit events → IPC → Preload → Store listeners → Vue components

### State Management (Pinia Stores)

Located in `src/store/`:

- **qso.ts**: Core QSO logging, station lookup, form state, WSJT-X decode handling
- **rig.ts**: Rig control state (frequency, mode, S-meter, split operation)
- **dxCluster.ts**: DX cluster spot management
- **propagation.ts**: Solar/propagation data
- **weather.ts**: Weather information
- **index.ts**: Basic counter store (minimal usage)

Stores are initialized in `App.vue` after setup wizard completes.

### Database Layer

**PouchDB** is used for local QSO storage (`src/services/DatabaseService.ts`):
- Database location: `userData/HamLedger.db` (managed by Electron's app.getPath)
- All QSO CRUD operations go through `DatabaseService`
- Main process handles all database operations; renderer communicates via IPC

### External Service Integration

**Services** (`src/services/`):
- **RigctldService.ts**: TCP connection to rigctld daemon for rig control (Hamlib)
- **WSJTXService.ts**: UDP server (port 2237) that receives WSJT-X binary protocol messages
- **QRZService.ts**: Fetches station data from QRZ.com XML API
- **OnlineStationService.ts**: Additional station lookup services
- **DatabaseService.ts**: PouchDB wrapper for QSO persistence

### Background Process Management

Main process manages two key background services:

1. **rigctld** (Hamlib): Started automatically on app launch, uses rig configuration from settings
2. **WSJT-X UDP Server**: Listens for decode messages and logged QSOs, enabled via settings

### Component Structure

- **App.vue**: Root component, handles initialization and setup wizard
- **MainContent.vue**: Main layout container
- **SideBar.vue**: Navigation sidebar
- **QsoPanel.vue**: Main QSO logging panel
- **LogBook.vue**: QSO history view
- **DxCluster.vue**: DX spot display
- **AppHeader.vue**: Contains sub-components:
  - **FreqSMeter.vue**: Frequency and S-meter display
  - **RigControl.vue**: Rig control interface
  - **PropClockWeather.vue**: Propagation, UTC clock, and weather
- **qso/** subdirectory:
  - **QsoInput.vue**: Main QSO entry form
  - **QsoDetailDialog.vue**: View QSO details
  - **QsoEditDialog.vue**: Edit existing QSO
  - **RemoteStation.vue**: Remote station information display

### Utility Modules

Located in `src/utils/`:

- **adif.ts**: ADIF file format parsing and generation
- **callsign.ts**: Callsign parsing and validation
- **maidenhead.ts**: Maidenhead grid square calculations
- **distance.ts**: Distance calculations between stations
- **bands.ts**: Amateur radio band definitions and frequency conversions
- **geocoding.ts**: Location geocoding utilities
- **dateHelper.ts**: UTC time formatting
- **smeterHelper.ts**: S-meter value conversions
- **configHelper.ts**: Configuration file management (settings.json)

### Configuration Management

Settings are stored in JSON format:
- **Default settings**: `src/settings.json` (bundled with app)
- **User settings**: `userData/settings.json` (platform-specific location)
- **Schema validation**: `src/settings.schema.json`

The `configHelper` utility manages loading, merging, and saving settings. Setup wizard creates initial user settings on first run.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [valibali/hamledger](https://github.com/valibali/hamledger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
