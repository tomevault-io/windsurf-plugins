---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## User-Facing Text

- Keep user-facing text in repo files (release notes, README) and the application (React UI) brief and to the point.
- Prefer the shortest complete wording; remove repeated context and optional detail unless it improves clarity.

## Contributor Guide

- For local setup, test commands, permission checks, and endpoint/storage gotchas, see `docs/CONTRIBUTING.md`.

## Project Overview

MemoryLane is a system tray Electron application that captures screenshots at regular intervals. Built with TypeScript using electron-vite for development and electron-builder for packaging. The application runs as a tray-only app (no main window) and captures screenshots based on user interaction and visual changes.

## Development Commands

```bash
# Start development mode (with hot reload)
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview

# Package the application (unpacked)
npm run package

# Create distributable packages
npm run make

# Platform-specific builds
npm run make:mac
npm run make:win

# Lint TypeScript files
npm run lint

# Format code with Prettier
npm run format

# Check formatting without writing (CI-friendly)
npm run format:check

# Run tests
npm run test
## E2E Tests

- Standalone E2E runners live in `e2e-tests/`.
- they will usually contain helpers in packge.json

## Architecture

### Application Structure

This is a tray-only application (no main window). The source follows electron-vite's directory convention:

```

src/
├── main/ # Main process
│ ├── index.ts # Entry point, tray management
│ ├── recorder/ # Screenshot capture module
│ ├── processor/ # OCR, embeddings, storage
│ └── mcp/ # MCP server integration
├── preload/ # Preload scripts
│ └── index.ts
├── renderer/ # Renderer process (minimal for tray app)
│ ├── index.html
│ ├── index.ts
│ └── index.css
└── shared/ # Shared types and constants
├── types.ts
└── constants.ts

````

### Native Sidecars (`native/`)

`native/` contains first-party native sidecar binaries (different from native Node.js addons in `node_modules`). Example is app-watcher module for windows.


### Main Components

1. **Main Process** (`src/main/index.ts`): Entry point and tray management
   - Creates system tray with menu (Start/Stop Capture, Capture Now, Quit)
   - Handles app lifecycle as a tray app (doesn't quit when windows close)
   - Hides dock icon on macOS for pure tray experience
   - Integrates with recorder and processor modules

2. **Recorder Module** (`src/main/recorder/`): Screenshot capture
   - Uses Electron's `desktopCapturer` API to capture screens
   - Saves screenshots as PNG files to `{userData}/screenshots/`
   - Event-driven capture based on user interaction and visual changes
   - Exposes API for downstream consumers via callback system

3. **Processor Module** (`src/main/processor/`): Screenshot processing
   - OCR using macOS Vision framework (Swift)
   - Vector embeddings using Transformers.js
   - Storage using SQLite (better-sqlite3 + sqlite-vec) for persistence, FTS, and vector search

### Build System

The project uses electron-vite for development and electron-builder for packaging:

- **electron.vite.config.ts**: Unified Vite configuration
  - Configures main, preload, and renderer builds
  - Handles native module externalization
  - Source maps enabled for debugging

- **electron-builder.config.js**: Packaging configuration
  - Defines targets for macOS and Windows
  - Configures ASAR unpacking for native modules
  - Handles asset copying to resources

### Editions

The app currently supports two editions:

- `customer`
- `enterprise`

Build selection is done with the `EDITION` environment variable in packaging scripts. The edition-specific build commands set it explicitly and `electron-builder.config.js` uses it to choose packaging metadata and bundle the matching `config/editions/<edition>.json` file.

At runtime, the app loads the edition config once during startup in `src/main/edition.ts`, validates it, and exposes the resolved config object to the rest of the app. In development, `EDITION` selects which config file to load; in packaged builds, the bundled `resources/config/edition.json` file is loaded.

### Native Modules

This project uses several native Node.js modules that require special handling:

- `uiohook-napi` - Keyboard/mouse monitoring
- `sharp` - Image processing
- `better-sqlite3` - SQLite database
- `onnxruntime-node` - ML inference

These are:

1. Externalized in Vite (not bundled)
2. Rebuilt for Electron via `postinstall` script
3. Unpacked from ASAR via electron-builder config

### Single-Runtime Environment

All scripts run under Electron's Node.js via `scripts/enode.sh`. Native modules are compiled once for Electron during `npm install` (via postinstall) and never need rebuilding. This eliminates ABI mismatch errors when switching between tests and dev.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deusXmachina-dev/memorylane](https://github.com/deusXmachina-dev/memorylane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
