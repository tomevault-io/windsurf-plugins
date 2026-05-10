---
trigger: always_on
description: **videOS** is a native macOS video player built entirely in **Swift + SwiftUI**, powered by **libVLC** for media playback. It is developed **without Xcode** — built, tested, and packaged entirely from the CLI using Swift Package Manager.
---

# videOS — Ultra-Advanced macOS Video Player

## Overview

**videOS** is a native macOS video player built entirely in **Swift + SwiftUI**, powered by **libVLC** for media playback. It is developed **without Xcode** — built, tested, and packaged entirely from the CLI using Swift Package Manager.

## Architecture

### Tech Stack
- **Language:** Swift 5.9+
- **UI Framework:** SwiftUI (macOS 13+)
- **Playback Engine:** libVLC 3.x via C interop
- **Persistence:** Codable + JSON (no CoreData/SwiftData dependency)
- **Build System:** Swift Package Manager + Make
- **Packaging:** Custom shell scripts → `.app` bundle

### libVLC Bridging Strategy

videOS bridges to libVLC's C API directly — no VLCKit, no Objective-C wrappers.

```
┌─────────────────────────────────────────────┐
│  SwiftUI Views                              │
│  ├── PlayerView (NSViewRepresentable)       │
│  ├── ControlBar                             │
│  └── LibraryView                            │
├─────────────────────────────────────────────┤
│  PlayerEngine (Swift class)                 │
│  ├── Wraps libvlc_media_player_t            │
│  ├── Publishes state via Combine            │
│  ├── C callback → Swift via pointer magic   │
│  └── Thread-safe: VLC callbacks on bg queue │
├─────────────────────────────────────────────┤
│  CLibVLC (SPM system library target)        │
│  ├── module.modulemap → libvlc headers      │
│  ├── shim.h → focused C type declarations   │
│  └── Links: -lvlc at build time             │
├─────────────────────────────────────────────┤
│  libvlc.dylib (from VLC.app or Homebrew)    │
└─────────────────────────────────────────────┘
```

**Key bridging details:**
- `CLibVLC/include/shim.h` declares opaque VLC types and the subset of functions we use
- `CLibVLC/module.modulemap` maps the shim to a Swift-importable module
- `PlayerEngine` uses `Unmanaged<PlayerEngine>` to pass `self` through C callbacks
- VLC events fire on background threads → dispatched to `@MainActor` via `DispatchQueue.main`
- NSView handed to VLC via `libvlc_media_player_set_nsobject()` for zero-copy rendering

### Module Structure

```
Sources/
├── videOS/                    # Main app target
│   ├── App.swift              # @main SwiftUI entry point
│   ├── AppDelegate.swift      # NSApplicationDelegate for menu/lifecycle
│   ├── Models/                # Pure data types (Codable)
│   ├── Services/              # Business logic, VLC integration
│   ├── ViewModels/            # Observable state for views
│   ├── Views/                 # SwiftUI views
│   │   └── Components/        # Reusable UI components
│   └── Utilities/             # Formatters, helpers
└── CLibVLC/                   # C bridging module for libVLC
    ├── include/shim.h         # libVLC type/function declarations
    └── module.modulemap       # Swift module map
```

## Build & Run

### Prerequisites

```bash
# Install VLC (provides libvlc.dylib)
brew install --cask vlc

# Or install libVLC headers + lib directly
brew install vlc
```

### Commands

```bash
make deps          # Install/verify dependencies
make build         # Debug build
make release       # Optimized release build
make run           # Build and launch
make package       # Create videOS.app bundle
make test          # Run tests
make clean         # Clean build artifacts
```

### How Linking Works

The build finds libVLC in these locations (in order):
1. `/Applications/VLC.app/Contents/MacOS/lib/`
2. `/usr/local/lib/` (Homebrew Intel)
3. `/opt/homebrew/lib/` (Homebrew Apple Silicon)
4. Custom `VLC_LIB_PATH` environment variable

## UI/UX — Glass Morphism Design

### Design Language
- **Glass panels** using SwiftUI `.ultraThinMaterial` / `.regularMaterial`
- **Vibrancy** text and icons on translucent surfaces
- **Dark-first** with automatic light mode support
- **Floating controls** that auto-hide during playback
- **Smooth animations** on all state transitions (spring-based)

### Layout

```
┌──────────────────────────────────────────────────┐
│ ◉ ◉ ◉  videOS            🔍  ☰               │ ← Title bar (transparent)
├────────┬─────────────────────────────────────────┤
│        │                                         │
│  📁    │                                         │
│ Library│         VIDEO CANVAS                    │
│        │       (NSView → VLC)                    │
│  🎵    │                                         │
│Playlists│                                        │
│        │                                         │
│  🌐    │                                         │
│Streams │                                         │
│        ├─────────────────────────────────────────┤
│  🔖    │   advancement ◀◀  ▶  ▶▶  🔊 ─── │ ← Control bar (glass)
│Bookmarks│  00:12:34 / 01:42:00  [A] [S]        │
├────────┴─────────────────────────────────────────┤
└──────────────────────────────────────────────────┘
```

### Key Interactions
- **Double-click** video → fullscreen toggle
- **Scroll** on video → volume
- **Scroll** on seek bar → scrub with thumbnail preview
- **Space** → play/pause
- **← →** → seek ±10s (shift: ±30s, alt: ±5s)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Worth-Doing/videOS](https://github.com/Worth-Doing/videOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
