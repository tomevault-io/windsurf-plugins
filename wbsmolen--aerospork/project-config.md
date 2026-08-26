---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

aerospork is an i3-like tiling window manager for macOS written in Swift. It uses macOS Accessibility APIs to manage windows in a tree-based layout paradigm. The project includes:
- A main application (aerosporkApp) that runs as a background service
- A CLI tool (`aerospork`) that communicates with the app via native Unix sockets
- TOML-based configuration system with hot-reload and a settings GUI
- DisplayLink-aware multi-monitor support

## Build Commands

### Debug Build
```bash
./build-debug.sh              # Build debug version to .debug/
./run-debug.sh                # Run the debug binary directly (.debug/aerosporkApp)
./build-debug-app.sh          # Assemble .debug/AeroSpork-Debug.app (needed for Accessibility)
./launch-debug-app.sh         # Launch that bundle
./run-cli.sh [args]           # Run aerospork CLI (forwards args to binary)
```

Debug builds use `~/.aerospork-debug.toml` instead of `~/.aerospork.toml` for configuration.

### Release Build
```bash
./build-release.sh            # Build release to .release/ using Xcode
./install-from-sources.sh     # Build and install as the aerospork-dev brew cask
```

### Testing & Code Quality
```bash
./run-tests.sh                # Run full test suite + lint checks
./run-swift-test.sh           # Run Swift tests only (swift test)
./format.sh                   # Format code with swiftformat + swiftlint
```

### Other Commands
```bash
./generate.sh                 # Regenerate generated files (*.xcodeproj, *Generated.swift)
./build-docs.sh               # Build site and man pages to .site/ and .man/
./build-shell-completion.sh   # Regenerate the vendored completions in shell-completion/
```

## Architecture

### Core Components

**Tree-Based Data Structure** (`Sources/AppBundle/tree/`)
- `TreeNode.swift`: Base class for all tree nodes (mutable double-linked structure)
- `Workspace.swift`: Top-level container representing a virtual workspace
- `TilingContainer.swift`: Container nodes with orientation (h/v) and layout (tiles/accordion)
- `MacWindow.swift`: Leaf nodes representing actual windows
- `MacApp.swift`: Application container nodes
- Tree uses parent-child relationships with MRU (most recently used) tracking

**Window Management Flow**
1. `GlobalObserver.swift` monitors NSWorkspace notifications (app launch/activate/hide/terminate)
2. Notifications trigger `runRefreshSession()` which synchronizes tree state with actual windows
3. Layout engine (`layout/layoutRecursive.swift`) calculates window positions/sizes
4. Refresh events are debounced (fixed 50ms) so bursts of accessibility notifications coalesce into a single layout pass

**Client-Server Architecture**
- `Sources/AppBundle/server.swift`: Unix socket server running in main app
- `Sources/Cli/_main.swift`: CLI client that connects to `/tmp/<bundle-id>-<user>.sock` — note the
  bundle id differs per build, so debug and release each have their own socket
  (`/tmp/com.wbs.aerospork.debug-<user>.sock` vs `/tmp/com.wbs.aerospork-<user>.sock`)
- IPC uses native POSIX `AF_UNIX` sockets with length-prefixed framing (`Sources/Common/util/UnixSocket.swift`), no third-party socket library
- Commands are parsed in `Sources/AppBundle/command/parseCommand.swift` and executed on server
- All commands implement the `Command` protocol

**Configuration System** (`Sources/AppBundle/config/`)
- `Config.swift`: Main config structure
- `parseConfig.swift`: the single canonical TOML parser (TOMLKit), mapping keys to `Config` via `WritableKeyPath`
- `ConfigurationWriter.swift`: comment-preserving writer used by the GUI (line-based, only rewrites UI-managed keys)
- `ConfigFileWatcher.swift`: hot-reload — watches the active config file and reloads on change
- Supports modes, hotkey bindings, workspace-to-monitor assignments, gaps, callbacks

> **Writer invariant — do not break this.** A section the user did not edit must be left byte for
> byte alone, so *saving without editing changes nothing*. The view model is a lossy projection of
> the config (per-monitor gap arrays collapse to their default, monitor fingerprints to a single
> string, monitor fallback lists to `.first`), so unconditionally re-serializing a section destroys
> whatever the UI can't express. This shipped as a real bug: a `{ fingerprint = { display_name,
> width, height } }` assignment was silently rewritten as a bare name regex. `ConfigurationWriter`
> guards every section on a `…Edited` flag from `ConfigurationViewModel`, and
> `testWriterNoOpSaveIsByteIdentical` enforces it.

**Layout System** (`Sources/AppBundle/layout/`)
- Recursive, synchronous layout algorithm in `layoutRecursive.swift`
- `MacApp.setFrame` skips redundant AX position/size writes when a window is already at the target frame (avoids framebuffer churn, important over DisplayLink/USB)
- Supports gaps (inner/outer), accordion padding, container orientation

**Hotkeys** (`Sources/AppBundle/config/`)
- Global hotkeys are registered with native Carbon `RegisterEventHotKey` (`HotkeyBinding.swift`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wbsmolen/aerospork](https://github.com/wbsmolen/aerospork) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
