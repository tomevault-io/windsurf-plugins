---
trigger: always_on
description: iOS SSH terminal using **libghostty** for GPU-accelerated rendering + **SwiftNIO SSH** for connections.
---

# Clauntty - iOS SSH Terminal with Ghostty

iOS SSH terminal using **libghostty** for GPU-accelerated rendering + **SwiftNIO SSH** for connections.

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  SwiftUI Views              Direct I/O          SwiftNIO SSH │
│  ┌──────────────┐         ┌───────────┐      ┌────────────┐  │
│  │ Terminal UI  │ ──────► │ SSH Data  │ ───► │ SSH Channel│  │
│  │ + Keyboard   │ ◄────── │ Flow      │ ◄─── │ (remote)   │  │
│  └──────────────┘         └───────────┘      └────────────┘  │
│         │                                          │         │
│         ▼                                          ▼         │
│  GhosttyKit.xcframework                     Remote Server    │
│  (Metal rendering)                                           │
└─────────────────────────────────────────────────────────────┘
```

**Data Flow**:
- **SSH → Terminal**: `SSHChannelHandler.channelRead()` → `ghostty_surface_write_pty_output()` → rendered
- **Keyboard → SSH**: `insertText()` → `SSHConnection.sendData()` → SSH channel

## Repository Layout

```
~/Projects/clauntty/
├── clauntty/          # iOS app (this repo)
├── ghostty/           # Forked ghostty (git@github.com:eriklangille/ghostty.git)
└── libxev/            # Local libxev fork (iOS fixes)
```

## Key Files

| Location | Purpose |
|----------|---------|
| `../ghostty/include/ghostty.h` | C API header |
| `../ghostty/src/termio/Exec.zig` | iOS process/PTY handling |
| `../ghostty/src/renderer/Metal.zig` | Metal renderer |
| `../libxev/src/backend/kqueue.zig` | Event loop (iOS fixes) |
| `Clauntty/Core/Terminal/` | GhosttyApp, TerminalSurface, GhosttyBridge |
| `Clauntty/Core/SSH/` | SSHConnection, SSHAuthenticator |
| `Clauntty/Core/Terminal/GhosttyApp.swift` | GhosttyApp + Logger extension with `debugOnly()`, `verbose()` |

## Build Commands

**Always use `./scripts/sim.sh` instead of raw `xcrun simctl` or `xcodebuild` commands.**

```bash
# Build GhosttyKit (after ghostty changes)
cd ../ghostty && zig build -Demit-xcframework

# Build & Run (use sim.sh)
./scripts/sim.sh build              # Build app
./scripts/sim.sh run                # Build, install, launch
./scripts/sim.sh debug devbox       # Full cycle: build, install, launch, screenshot, logs
./scripts/sim.sh quick devbox       # Skip build, just reinstall (faster iteration)

# Multi-Tab Debugging (last tab is active on launch)
./scripts/sim.sh debug devbox --tabs "0,1"    # 2 existing sessions (tab 2 active)
./scripts/sim.sh debug devbox --tabs "0,new"  # 1 existing + 1 new (new tab active)
./scripts/sim.sh tap-tab 1 2                   # Switch to tab 1 (of 2 total)
./scripts/sim.sh run-tab 1 2 "ls -la"         # Run command in tab 1

# Logs & Screenshots
./scripts/sim.sh logs 30s            # Show last 30 seconds
./scripts/sim.sh screenshot myshot   # Save screenshot

# See all commands
./scripts/sim.sh help

# Run tests
xcodebuild test -project Clauntty.xcodeproj -scheme ClaunttyTests \
  -destination 'platform=iOS Simulator,name=iPhone 17'

# Build for physical iPhone
xcodebuild -project Clauntty.xcodeproj -scheme Clauntty \
  -destination 'platform=iOS,name=iPhone 16' -quiet build

# Install and launch on iPhone
xcrun devicectl device install app --device "iPhone 16" \
  ~/Library/Developer/Xcode/DerivedData/Clauntty-*/Build/Products/Debug-iphoneos/Clauntty.app
xcrun devicectl device process launch --device "iPhone 16" com.octerm.clauntty
```

## TestFlight Upload

Archive and upload to App Store Connect for TestFlight distribution:

```bash
# 1. Clean and archive
rm -rf build
xcodebuild -project Clauntty.xcodeproj -scheme Clauntty clean -quiet
xcodebuild -project Clauntty.xcodeproj -scheme Clauntty \
  -destination 'generic/platform=iOS' \
  -archivePath build/Clauntty.xcarchive archive \
  -allowProvisioningUpdates

# 2. Export and upload to App Store Connect
xcodebuild -exportArchive \
  -archivePath build/Clauntty.xcarchive \
  -exportOptionsPlist ExportOptions.plist \
  -exportPath build/export \
  -allowProvisioningUpdates
```

**Prerequisites:**
- Distribution certificate: "Apple Distribution: Octerm Technologies, Inc."
- App created in App Store Connect with bundle ID `com.octerm.clauntty`
- `ExportOptions.plist` in project root (team ID: 65533RB4LC)

**After upload:**
1. Go to [appstoreconnect.apple.com](https://appstoreconnect.apple.com)
2. Select Clauntty → TestFlight
3. Wait for build processing (5-30 min)
4. Add testers (internal = instant, external = requires review)

## Logging & Debugging

### Log Levels

The app uses a tiered logging system optimized for performance:

| Method | When Logged | Use Case |
|--------|-------------|----------|
| `Logger.clauntty.error()` | Always | Errors, failures |
| `Logger.clauntty.warning()` | Always | Warnings |
| `Logger.clauntty.debugOnly()` | DEBUG builds only | Lifecycle events, state changes |
| `Logger.clauntty.verbose()` | DEBUG + CLAUNTTY_VERBOSE=1 | Per-packet, per-frame, per-touch |

**When to use each level:**
- **`error/warning`**: Problems that need attention
- **`debugOnly()`**: Session lifecycle, tab switching, connection events, one-time init - things you want during normal debugging

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eriklangille/clauntty](https://github.com/eriklangille/clauntty) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
