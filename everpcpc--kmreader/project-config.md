---
trigger: always_on
description: This file provides guidance to coding agents working with code in this repository.
---

# AGENTS.md

This file provides guidance to coding agents working with code in this repository.

## Project Overview

**KMReader** is a native SwiftUI client for [Komga](https://github.com/gotson/komga), a self-hosted digital comic/book library manager. The app supports iOS 17.0+, macOS 14.0+, and tvOS 17.0+ with Swift 6.0+ and Xcode 15.0+.

### Readers

- **DIVINA Reader** (iOS, macOS, tvOS): LTR/RTL/vertical/Webtoon modes with spreads, zoom, customizable tap zones, and page curl transitions. Live Text support with shake-to-toggle on iOS.
- **EPUB Reader** (iOS, macOS): Native engine with custom font importing (.ttf/.otf), theme presets, multi-column layouts, and nested TOC navigation.
- **Per-Book Preferences**: Save reading direction, page layout, and theme settings per book.
- **Incognito Mode**: Read without saving progress to server.

### Offline & Downloads

- **Background Downloads**: URLSession-based downloads with Live Activities on iOS.
- **Series Policies**: Manual, unread-only, unread+cleanup, or all books per series.
- **Offline Mode**: Full reader functionality with downloaded content. Progress syncs when reconnected.
- **Two-Tier Caching**: Pages and thumbnails with adjustable limits and auto-cleanup.

### Browse & Dashboards

- **Dynamic Dashboards**: Keep Reading, On Deck, Recently Added, Recently Updated with real-time SSE updates.
- **Advanced Filters**: Search with metadata filters (authors, genres, tags, publishers) using all/any logic.
- **Grid/List Layouts**: Multiple density options (compact, standard, comfortable).
- **Library Filtering**: Browse per-library or across all libraries.

### Multi-Server Vault

- **Unlimited Servers**: Save multiple Komga instances with password or API key authentication.
- **Quick Switching**: Instant server switching with isolated data per instance.
- **API Key Management**: Create, view, and revoke API keys.

### Admin Tools

- **Metadata Editing**: Edit series, books, collections, and read lists.
- **Library Management**: Create, edit, scan libraries with directory browser.
- **Task Management**: Monitor and cancel server tasks with live metrics.
- **Logs Viewer**: View and export app logs with filtering.

### Platform-Specific

- **iOS**: Live Activities, background downloads, page curl transitions, shake gestures.
- **macOS**: Separate reader windows, comprehensive keyboard shortcuts, keyboard help overlay.
- **tvOS**: Remote control navigation, TV-optimized interface (DIVINA only).

## Commands

### Build Commands

All build and run commands use `misc/xcode.py` internally. The script manages device selection and persists preferences in `devices.json`.

For iOS and tvOS builds, the script will select a specific simulator (using saved preference or prompting for selection).

```bash
# Build for specific platforms
# Builds for a specific simulator (will use saved preference or prompt for selection)
make build-ios          # Build for iOS simulator
make build-macos        # Build for macOS
make build-tvos         # Build for tvOS simulator

# Other build commands
make build              # Build all platforms
make release            # Archive and export all platforms
make clean              # Remove archives and exports
```

Build execution rule:
- Do not run multiple `make build-*` commands in parallel. `xcodebuild` shares the same DerivedData build database and parallel runs may fail with database lock errors.
- Prefer `make build` for full validation.
- If platform-specific builds are required, run `make build-ios`, `make build-macos`, and `make build-tvos` sequentially.

### Run Commands

Run commands support device selection with preferences stored in `devices.json`.

```bash
# List available devices (simulators and physical devices)
make list-device

# Run on simulators
make run-ios-sim        # iOS simulator
make run-tvos-sim       # tvOS simulator

# Run on physical devices
make run-ios-device     # iOS device
make run-tvos-device    # tvOS device

# Run on macOS
make run-macos          # Build and run on macOS

# Force device selection (ignore saved preference)
make run-ios-sim-select      # iOS simulator with device selection prompt
make run-ios-device-select   # iOS device with device selection prompt
make run-tvos-sim-select     # tvOS simulator with device selection prompt
make run-tvos-device-select  # tvOS device with device selection prompt

# Direct script usage (alternative to make commands)
python3 misc/xcode.py list                    # List all devices
python3 misc/xcode.py list ios --simulators   # List iOS simulators only
python3 misc/xcode.py build ios               # Build for iOS
python3 misc/xcode.py run ios --simulator     # Run on iOS simulator
python3 misc/xcode.py run ios --device        # Run on iOS device
python3 misc/xcode.py run ios --simulator --select  # Force device selection
```

Device selection behavior:
- **Interactive mode** (terminal): Prompts you to select from available devices and optionally save as default
- **Non-interactive mode** (CI/scripts): Automatically selects the first available device and saves it
- **Saved preference**: If a device is already saved in `devices.json`, it will be used automatically

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [everpcpc/KMReader](https://github.com/everpcpc/KMReader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
