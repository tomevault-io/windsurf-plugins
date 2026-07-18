---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Meteor WebGUI is a Meteor Client addon that provides a web-based GUI with real-time bi-directional control. The project consists of two main components:
- **Java Backend**: Fabric mod addon that creates a WebSocket server for communication
- **Vue Frontend**: Modern web interface for controlling Meteor Client modules and settings

## Essential Development Commands

### Java/Gradle Build
```bash
# Build the addon JAR
./gradlew build

# Run in Minecraft dev environment
./gradlew runClient

# Run tests
./gradlew test

# Clean build artifacts
./gradlew clean
```

Output JAR: `build/libs/meteor-webgui-0.3.0.jar`

### WebUI Development
```bash
cd webui

# Install dependencies (first time)
npm install

# Development server with hot-reload (connects to ws://localhost:8080)
npm run dev
# Access at: http://localhost:3000

# Production build
npm run build
# Output: webui/dist/
```

### Testing the Full Stack

**Development Mode (with Vite hot-reload):**
1. Start Minecraft with the addon: `./gradlew runClient`
2. In-game: Open Meteor GUI (Right Shift) → WebGUI tab → Start Server
3. In separate terminal: `cd webui && npm run dev`
4. Open browser: `http://localhost:3000` (Vite dev server)

**Production Mode (served from addon):**
1. Build WebUI: `cd webui && npm run build`
2. Start Minecraft: `./gradlew runClient`
3. In-game: Open Meteor GUI (Right Shift) → WebGUI tab → Start Server
4. Open browser: `http://localhost:8080` (served from addon's HTTP server)

## Key Architecture Concepts

### Bi-Directional Sync Architecture
The addon maintains real-time synchronization between Minecraft and the web interface:
- **HTTP Server**: Serves the built WebUI from `webui/dist/` on port 8080 (production mode)
- **WebSocket Server**: Real-time bidirectional communication on port 8080 (same port, different protocol)
- **Java → WebUI**: Event monitoring broadcasts module/setting changes via WebSocket
- **WebUI → Java**: User interactions send commands to modify game state
- **Initial State**: Full module/settings snapshot sent on WebSocket connection
- **HUD Preview**: Periodic snapshots of HUD elements sent to WebUI for real-time preview

### Module Mapping System
The `ModuleMapper` class (src/main/java/com/cope/meteorwebgui/mapping/) dynamically discovers ALL modules from Meteor Client and installed addons at runtime. It does NOT require manual registration - it automatically iterates through `Modules.get().getAll()` and maps each module by category.

### HUD Mapping System
The `HudMapper` class provides dynamic discovery and preview of HUD elements:
- Iterates through all HUD elements from Meteor Client and addons
- Captures real-time text content from each HUD element
- Generates preview snapshots showing what will be rendered in-game
- Supports toggling HUD element visibility from the WebUI
- Uses mixins to hook into HUD lifecycle and rendering events

### Settings Reflection System
The `SettingsReflector` class provides generic read/write access to any Meteor setting type via reflection. It:
- Detects setting types using `setting.getClass().getSimpleName()`
- Extracts type-specific metadata (e.g., min/max for IntSetting, enum values for EnumSetting)
- Handles type conversion for JSON serialization/deserialization
- Uses `setting.get()` for reads and `setting.set(T)` for writes

See `ai_docs/METEOR_SETTINGS_RESEARCH.md` for comprehensive details on all 30+ setting types.

### Event Monitoring Pattern
Real-time sync is achieved by wrapping existing Meteor event handlers:
- `ActiveModulesChangedEvent`: Detects module toggles
- Setting `onChanged` callbacks: Wrapped to broadcast setting changes
- All events broadcast to connected WebSocket clients via `WSMessage` protocol

### WebSocket Protocol
Message format defined in `src/main/java/com/cope/meteorwebgui/protocol/`:
```typescript
interface WSMessage {
    type: string;  // e.g., "module.toggle", "setting.update", "initial.state"
    data: any;
    id?: string;   // Request ID for request/response pattern
}
```

**Server → Client Events:**
- `initial.state`: Full module/settings state on connection
- `module.state.changed`: Module toggled
- `setting.value.changed`: Setting value changed
- `hud.snapshot`: HUD preview snapshot with rendered text lines
- `hud.list`: List of all available HUD elements

**Client → Server Commands:**
- `module.toggle`: Toggle module on/off
- `setting.update`: Update setting value
- `module.list`: Request full module list
- `hud.list`: Request HUD elements list
- `hud.toggle`: Toggle HUD element visibility

## Critical References

### ai_reference Folder
**IMPORTANT**: The `ai_reference/` folder (git-ignored) contains high-quality example sources for Meteor Client addon development. **ALWAYS read `ai_reference/INDEX.md` first** when working on Meteor-specific features.

The INDEX.md file contains:
- Complete catalog of all reference repositories
- Quick lookup guide for specific features (modules, settings, commands, HUD, etc.)
- Repository details with star counts, update dates, and feature lists
- Usage guidelines and research strategies

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MCDxAI/meteor-client-webgui](https://github.com/MCDxAI/meteor-client-webgui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
