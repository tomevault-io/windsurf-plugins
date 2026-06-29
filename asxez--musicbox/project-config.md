---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

There's a file modification bug in Claude Code. The workaround is: always use complete absolute Windows paths with drive letters and backslashes for ALL file operations. Apply this rule going forward, not just for this file.

## CRITICAL: File Editing on Windows

### Always Use Backslashes on Windows for File Paths

**When using Edit or MultiEdit tools on Windows, you MUST use backslashes (`\`) in file paths, NOT forward slashes (`/`).**

```
WRONG:  Edit(file_path: "D:/repos/project/file.tsx", ...)
RIGHT:  Edit(file_path: "D:\repos\project\file.tsx", ...)
```

## Project Overview

MusicBox is a plugin-based local music player built with Electron. It supports multiple audio formats (flac, mp3, wav, ogg, m4a, aac, wma) and features a plugin system inspired by VSCode's extension architecture.

## Technology Stack

- **Main Process**: Electron (v41.2.1) with TypeScript, Node.js (>=24.15.0)
- **Renderer Process**: Vite + TypeScript/vanilla JS hybrid (migrating to TypeScript)
- **Native Audio**: Rust (v1.94.1) with WASAPI exclusive mode support, built via napi-rs
- **Python**: Used for metadata editing utilities (>=3.8), compiled to `.exe` via PyInstaller
- **Styling**: SCSS

## Development Commands

```bash
# Install all dependencies
npm install && npm run install:renderer && npm run install:rs
pip install -r requirements.txt

# Development (builds renderer + Rust + TypeScript, then launches Electron with --expose-gc)
npm run dev

# Build renderer only (for UI iteration)
npm run dev:renderer    # Vite dev server on port 8080

# Build TypeScript (main process) only
npm run build:ts        # Compiles src/main/**/*.ts to dist/main/
npm run watch:ts        # Watch mode for TypeScript compilation

# Build Rust native module only
npm run build:rs

# Build Python metadata editor only
npm run build:python    # Compiles metadata_editor.py to .exe via PyInstaller

# Type-check renderer (separate from build)
npm run typecheck:renderer

# Full application build
npm run build

# Platform-specific builds
npm run build:win       # Windows (NSIS + portable)
npm run build:mac       # macOS (DMG + ZIP)
npm run build:linux     # Linux (AppImage + deb + rpm)

# Lint renderer code
cd src/renderer && npm run lint

# Clean build artifacts
npm run clean
```

Note: `npm run dev` is NOT hot-reload -- it runs a full `build:renderer` + `build:rs` + `build:ts` before launching Electron. For fast UI iteration, use `dev:renderer` separately.

## Architecture

### Multi-Process Structure

```
Main Process (Node.js + TypeScript)   <-> IPC <->   Renderer Process (Chromium)
├── Application class (core/Application.ts)         ├── Vite + TS/JS hybrid
├── Controllers (IPC handlers via decorators)       ├── Plugin system (TypeScript)
├── Services (library, network, extensions)         ├── Feature modules
├── WindowManager & ConfigManager                   ├── UI layer (base/dialogs/modals/pages/widgets)
├── Native audio engine (Rust N-API)                ├── API layer (MusicBoxAPI)
├── Tray, HTTP server, global shortcuts             ├── Infrastructure & shared utilities
└── Python metadata editor (subprocess)             └── Desktop lyrics window (separate HTML entry)
```

### Main Process (`src/main/`)

**Architecture Pattern**: Controller-based with dependency injection

- **`main.ts`**: Entry point -- creates `Application` instance and manages app lifecycle
- **`core/Application.ts`**: Central orchestrator -- initializes services, registers controllers, manages startup. All 22 controllers are instantiated and registered in `registerStartupControllers()`.
- **`core/ServiceContainer.ts`**: Dependency injection container for services
- **`core/WindowManager.ts`**: Window creation and management (main window, mini mode, etc.)
- **`core/ConfigManager.ts`**: Configuration file management
- **`preload.ts`**: Preload script bridging main/renderer via contextBridge. Exposes namespaced APIs: audio, nativeAudio, library, settings, lyrics, covers, networkDrive, window, desktopLyrics, tray, globalShortcuts, extensions, userdata, memory, benchmark, httpServer, app, hardwareAcceleration, and file operations.
- **`controllers/`**: 22 IPC handler classes using `@IpcHandler` decorator pattern. Key controllers: AudioController, NativeAudioController, LibraryController, NetworkController, LyricsController, CoversController, DesktopLyricsController, WindowController, TrayController, HttpServerController, GlobalShortcutsController, ExtensionsController, SettingsController, HardwareAccelerationController, BenchmarkController, MemoryController, UserDataController, AppController, DialogController, FileController, SystemController.
- **`services/`**: Business logic
  - `library/`: LibraryCacheManager, MetadataHandler, AutoScanScheduler
  - `network/`: NetworkDriveManager, NetworkFileAdapter, DriveRegistry
  - `extensions/`: ExtensionInstaller
- **`decorators/IpcHandler.ts`**: Decorator for automatic IPC handler registration
- **`utils/`**: Utility functions (metadata parsing, path security, file search)
- **`types/`**: TypeScript type definitions

### Renderer Process (`src/renderer/src/`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [asxez/MusicBox](https://github.com/asxez/MusicBox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
