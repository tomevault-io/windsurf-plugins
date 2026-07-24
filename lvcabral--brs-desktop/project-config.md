---
trigger: always_on
description: This is an Electron-based desktop application that simulates Roku devices for BrightScript development. It wraps the `brs-engine` library (imported as `brs-engine` npm package) and provides a complete Roku device simulator with networking services.
---

# BrightScript Simulator Desktop - AI Coding Instructions

## Project Overview
This is an Electron-based desktop application that simulates Roku devices for BrightScript development. It wraps the `brs-engine` library (imported as `brs-engine` npm package) and provides a complete Roku device simulator with networking services.

## Architecture

### Electron Process Structure

#### Main Process (`src/main.js`)
- **Application Lifecycle**: Electron app initialization, window creation, global state management
- **Device Information**: Creates unified `deviceInfo` object with Roku device specs, network config, localization
- **Server Orchestration**: Initializes and manages ECP, Installer, and Telnet servers
- **Menu System**: Platform-specific menu creation and IPC event routing
- **Settings Integration**: Loads/applies user preferences from JSON storage
- **Command Line Processing**: Handles startup arguments (devtools, console, files, etc.)
- **Global Shared State**: `globalThis.sharedObject` for cross-process data sharing

#### Main Process Helper Modules (`src/helpers/`)
- **`settings.js`**: ElectronPreferences integration, device configuration, UI themes
- **`window.js`**: Window management (create, focus, aspect ratio, screenshot, fullscreen)
- **`files.js`**: File loading (ZIP/BPK packages, BRS source), recent files management
- **`console.js`**: Telnet server integration, debug message routing
- **`dialog.js`**: Native file dialogs (open packages, save screenshots)
- **`about.js`**: About window with version information
- **`util.js`**: Network utilities (local IPs, gateway detection)
- **`roku.js`**: Peer Roku device communication via ECP

#### Main Process Menu System (`src/menu/`)
- **`menuService.js`**: Central menu management, recent files, context menus
- **`*MenuTemplate.js`**: Platform-specific menu definitions (File, Edit, Device, View, Help)
- **`macOSMenuTemplate.js`**: macOS-specific application menu structure

#### Renderer Process (`src/app/app.js`)
- **BRS Engine Interface**: Global `brs` object initialization and event subscription
- **Device Simulation UI**: Display management, stats overlay, theme handling
- **App Lifecycle Events**: Handles loaded/started/closed/error events from engine
- **Input Management**: Keyboard/gamepad mapping, custom key bindings
- **IPC Communication**: Main ↔ Renderer messaging via preload bridge
- **Debug Integration**: Micro debugger support, console redirection

#### Renderer Process Modules (`src/app/`)
- **`preload.js`**: Secure contextBridge API for main ↔ renderer communication
- **`statusbar.js`**: Bottom status bar (file info, services, resolution, audio)
- **`editor.js`**: CodeMirror-based BrightScript code editor window
- **`brightscript.js`**: CodeMirror syntax highlighting for BrightScript language
- **`codemirror.js`**: CodeMirror configuration and theme management

### Core Components

#### BRS Engine Integration
- **Global `brs` Object**: Exposes `initialize()`, `subscribe()`, `deviceData`, `getVersion()`, `getSerialNumber()`
- **Event System**: Engine publishes events (loaded, started, closed, error, debug, redraw, control)
- **Device Data Sync**: `brs.deviceData` properties sync with main process settings
- **Custom Key Mapping**: Supports Roku remote buttons + game controller inputs
- **Performance Stats**: Optional overlay showing FPS, memory, draw calls

#### Network Services (All run in main process)
- **ECP Server** (`src/server/ecp.js`, port 8060): 
  - REST API: `/query/device-info`, `/query/apps`, `/keypress/*`, `/launch/*`
  - ECP-2 WebSocket API for mobile app compatibility
  - SSDP discovery service for device detection
  - Observer pattern for event distribution
- **Web Installer** (`src/server/installer.js`, default port 80):
  - HTTP digest authentication (username: rokudev)
  - File upload interface for ZIP/BPK deployment
  - Screenshot capture and download
  - Channel deletion and management
- **Telnet Server** (`src/server/telnet.js`, port 8085):
  - Remote console access for debugging
  - Command execution and output streaming
  - Micro debugger integration
  - Multi-client support with observer pattern

#### Settings Architecture
- **Storage**: JSON file in `app.getPath("userData")/brs-settings.json`
- **UI**: `@lvcabral/electron-preferences` with custom CSS themes
- **Structure**: Nested sections (simulator, services, device, display, remote, audio, localization, captions)
- **Dot Notation Access**: `settings.value("device.deviceModel")` for nested properties
- **Live Updates**: Settings changes trigger IPC events to update running simulation

### Build System
- **Webpack**: Multi-config build in `build/webpack.app.config.js` 
  - Main entry: Creates `app/main.js` from `src/main.js`
  - App entry: Creates `app/app.js` from `src/app/app.js` 
  - Editor entry: Creates `app/editor.js` from `src/app/editor.js`
- **Development**: `npm run start` runs `build/start.js` with webpack watch + electron spawn
- **Release**: `npm run dist` builds production bundles + electron-builder packages

## Key Patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lvcabral/brs-desktop](https://github.com/lvcabral/brs-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
