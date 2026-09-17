---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Development Commands

```bash
# Development
npm run dev              # Launch Electron app in development mode

# Testing
npm test                 # Run all unit tests (Vitest)
npm run test:watch       # Run tests in watch mode
npm run test:e2e         # Run E2E tests (Playwright)
npm run test:e2e:ci      # Run E2E tests without Electron smoke test

# Building
npm run build            # Build renderer processes with Vite
npm run pack             # Package app without creating installer
npm run dist             # Create distributable installer
```

## Development Policy

- Add focused debug logging at key decision points when implementing or changing behavior, especially around IPC boundaries, asset import/transcoding, rule matching, runtime state transitions, animation playback, and error/fallback paths. Prefer the existing logger helpers and include enough structured context to diagnose issues without reproducing them blindly.
- This project is currently in active development. Do not spend effort preserving backward compatibility for existing local data, manifests, configs, or package formats unless explicitly requested. Prefer simple, correct data shapes and migrations-by-reset over compatibility layers.

## Architecture Overview

### Electron Process Model

This is a **three-process Electron application** with strict security boundaries:

1. **Main Process** (`src/main/`)
   - Entry point: `main.js` - initializes config, windows, IPC, tray
   - Window management: `windows.js` - pet window (320x320, transparent, frameless, always-on-top) and panel window (960x680, standard frame)
   - IPC hub: `ipc.js` - 31 registered channels handling config, assets, petpack, display, system settings
   - Services: `config-store.js`, `asset-store.js`, `petpack.js`

2. **Preload Scripts** (`src/preload/`)
   - Security bridge using `contextBridge` with context isolation
   - `pet-preload.js` exposes `window.desktopPet` API
   - `panel-preload.js` exposes `window.desktopPetPanel` API
   - No direct Node.js access from renderer processes

3. **Renderer Processes** (`src/renderer/`)
   - Pet renderer: sprite animation, event handling, rule engine integration
   - Panel renderer: modular control panel UI (see Panel Architecture below)
   - Built with Vite for production

### Panel Architecture (Modular)

The control panel (`src/renderer/panel/`) is modular. Current long-lived tabs are:

- `overview.js` - Overview dashboard, runtime status, recent events
- `assets.js` - Asset management and petpack import/export
- `animations.js` - Default animation, clips, keyframes, green screen settings
- `rules.js` - Trigger rules, inline actions, exit conditions/actions, conflict warnings
- `display.js` - Display settings
- `system.js` - System settings, language, logs

Legacy States/Actions tabs are no longer registered in the tab navigation. The current model is `assets -> animations -> triggerRules -> inline actions`.

**Main Entry**:
- `panel.js` - Initializes DOM/API references, loads config, routes events, coordinates rendering, polls runtime state

**State Management**:
- `state.js` - Global panel state container
- `panel-state.js` - Config operations, form model helpers, ID generation, keyframe normalization

**UI Layer**:
- `ui/banner.js` - Banner message management
- `ui/header.js` - Common header component
- `ui/tabs.js` - Current tab navigation
- `ui/utils.js` - HTML utilities and form helpers

**Reusable Components**:
- `inline-action-editor.js` - Scoped action editor for rule actions and exit actions
- `keyframe-editor.js` - Keyframe input/output mapping editor
- `rule-condition-editor.js` - Rule condition editor
- `rule-action-selector.js` - Action selector
- `condition-browser.js` - Condition type browser

**Event Handlers**:
- `form-handlers.js` - Form submission and form-to-config parsing
- `asset-handlers.js` - Asset and petpack operations
- `event-handlers.js` - Click, change, input, drag/drop sorting, modal interactions

### Petpack System

**Petpack** is the `.petpack` file format (ZIP) containing:
- `manifest.json` - package metadata, animations, trigger rules
- Asset files (GIF, WebP, WebM, MP4, MOV, PNG, SVG)

**Import flow** (`src/main/services/petpack.js`):
1. Validation: check ZIP structure, paths, and supported file types
2. Security: reject symlinks and unsafe paths; stream extraction with dynamic memory/disk resource guards (no fixed per-file, per-package, or entry-count limits)
3. Manifest validation against schema (`src/shared/manifest-validator.js`)
4. Atomic installation: extract to temp → backup existing → atomic rename → rollback on failure

**Export flow**:
1. Collect package files recursively
2. Merge manifest with user config (animations, trigger rules, interactions)
3. Validate merged manifest
4. Create ZIP with merged manifest, atomic write to target

**Storage structure**: `{userDataDir}/packages/{packageId}/manifest.json` and `assets/`

### Rule Runtime

**Core files**:
- `src/shared/rule-engine.js` - pure rule matching helpers
- `src/renderer/pet/pet-runtime.js` - runtime model building, event history, cooldown state

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duzexu/desktop-pet](https://github.com/duzexu/desktop-pet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
