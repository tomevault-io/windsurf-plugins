---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Audio Remote is a native macOS menu bar app for controlling both microphone (input) and speaker (output) audio with iOS Shortcuts support. This is a Swift rewrite of a previous Python implementation, achieving 50x faster toggle latency (~1ms vs ~50ms).

**Key Technologies:**
- Swift 5.9+ with SwiftUI for UI
- Core Audio API for direct audio device control (input & output)
- Vapor framework for HTTP server
- Custom GitHub Releases integration for auto-updates
- Combine for reactive state management

## Build & Run Commands

```bash
# Development build
swift build
.build/debug/AudioRemote

# Release build
swift build -c release
.build/release/AudioRemote

# Build app bundle
./scripts/build_app_bundle.sh

# Open in Xcode
open Package.swift
```

## Testing HTTP Server

```bash
# Microphone control (with confirmation - waits for Chrome Extension)
curl -X POST http://localhost:8765/toggle-mic

# Microphone control (fast mode - no waiting)
curl -X POST http://localhost:8765/toggle-mic/fast

# Check status
curl http://localhost:8765/status

# Volume control
curl -X POST http://localhost:8765/volume/increase
curl -X POST http://localhost:8765/volume/decrease
curl -X POST http://localhost:8765/volume/toggle-mute

# Set specific volume (0.0-1.0)
curl -X POST http://localhost:8765/volume/set -H "Content-Type: application/json" -d '{"volume": 0.5}'

# Web UI
open http://localhost:8765

# Test confirmation flow
./scripts/test_confirmation.sh
```

## Release Process

**Fully automated** - zero input required:

```bash
# From Claude Code
/release

# From command line
./scripts/release_auto.sh
```

**What it does automatically:**
1. Auto-commits any uncommitted changes
2. Auto-detects version bump from commit messages (MAJOR/MINOR/PATCH)
3. Auto-generates release notes from git history
4. Builds Rust FFI + Swift app
5. Creates DMG and ZIP
6. Commits, tags, and pushes
7. Creates GitHub Release

**Commit conventions for auto-versioning:**
- `breaking:` → MAJOR bump (3.0.0)
- `feat:` or `✨` → MINOR bump (2.8.0)
- `fix:` or `🔧` → PATCH bump (2.7.2)
- Others → PATCH bump

**No signing or appcast.xml needed** - app uses custom GitHub Releases integration.

### Update System Critical Guidelines

**ALWAYS test updates with PREVIOUS version's app, not new build!**

```bash
# ❌ WRONG: Testing new → new
./build/release/AudioRemote.app  # Check for Updates

# ✅ CORRECT: Testing old → new
/Applications/AudioRemote.app    # Check for Updates
```

**String interpolation in shell commands:**
```swift
// ❌ WRONG: Creates literal '\(var)' string
shell("command '\\(variable)'")

// ✅ CORRECT: Interpolates variable value
shell("command '\(variable)'")
```

**Emergency: Remove problematic asset from release:**
```bash
# If DMG install fails, force ZIP usage
gh release delete-asset vX.X.X AudioRemote-X.X.X.dmg --yes
```

**See `docs/UPDATE_TROUBLESHOOTING.md` for full troubleshooting guide.**

## Architecture

### Core Components

| Component | File | Responsibility |
|-----------|------|----------------|
| BridgeManager | `Core/BridgeManager.swift` | Chrome Extension bridge with confirmation pattern |
| HTTPServer | `Core/HTTPServer.swift` | Vapor-based async HTTP server with REST endpoints |
| SettingsManager | `Core/SettingsManager.swift` | UserDefaults persistence, legacy migration |
| UpdateManager | `Core/UpdateManager.swift` | Custom GitHub Releases update integration |
| MenuBarController | `UI/MenuBarController.swift` | NSStatusItem, NSPopover, icon updates |
| GlobalHotkeyManager | `Core/GlobalHotkeyManager.swift` | System-wide keyboard shortcuts |
| AppDelegate | `App/AppDelegate.swift` | App lifecycle, manager initialization |

### App Initialization Flow

1. `AudioRemoteApp` → `AppDelegate.applicationDidFinishLaunching`
2. `NSApp.setActivationPolicy(.accessory)` (hide dock icon)
3. Initialize managers: BridgeManager → SettingsManager → UpdateManager → MenuBarController → HTTPServer
4. Start HTTP server if enabled in settings
5. Set up Combine publishers for reactive updates

### Chrome Extension Bridge Architecture

**MicDrop now operates as a bridge between iOS Shortcuts and Chrome Extension:**

```
iOS Shortcuts → MicDrop Server → Chrome Extension → Google Meet
                      ↑                    ↓
                      └─── Confirmation ───┘
```

**Confirmation Pattern:**
- `/toggle-mic` (default): Waits for Chrome Extension to confirm successful mute (3s timeout)
- `/toggle-mic/fast`: Legacy optimistic toggle (no waiting)
- Extension polls `/bridge/poll` for events (long-polling)
- Extension POSTs actual state to `/bridge/mic-state` after mute

**Key Benefits:**
- ✅ Guarantees mute actually happened in Google Meet
- ✅ Returns timeout if extension not running
- ✅ Syncs actual state from Meet UI, not assumed state
- ✅ ~100ms latency with confirmation vs ~1ms fast mode

**Documentation:**
- Full guide: `docs/EXTENSION_INTEGRATION.md`
- Example extension: `examples/chrome-extension/`
- Test script: `scripts/test_confirmation.sh`

### State Management

- All managers are `ObservableObject` with `@Published` properties

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leolionart/MicDrop](https://github.com/leolionart/MicDrop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
