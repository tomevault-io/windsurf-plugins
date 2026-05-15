---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ZenDevToolkit is a lightweight macOS menu bar application that provides developers with quick access to commonly-needed utility functions without opening a web browser. The app lives in the system menu bar and opens a clean, organized popup interface (360x500px) when clicked.

**Target Platform**: macOS native app for Mac App Store distribution ($3.99 one-time purchase)
**Target Users**: Software developers who frequently need to format JSON, encode/decode data, generate hashes, and perform other common development tasks

## Architecture

### Core Components

- **ZenDevToolkitApp.swift**: Main app entry point with menu bar setup via AppDelegate
  - Manages NSStatusItem for menu bar presence
  - Controls NSPopover for tool interface (360x500 size)
  
- **ContentView.swift**: Main UI container that hosts all tools
  - Segmented picker for tool selection
  - Currently implements JSON formatter with placeholder views for other tools

### Tool Views

Each tool is implemented as a separate SwiftUI view:
- JSONFormatterView: Fully implemented with format/minify/validate functionality
- TimestampConverterView: Fully implemented with Unix/human date conversion and timezone support
- Base64View, URLEncoderView, HashGeneratorView, UUIDGeneratorView: Placeholder implementations

## Build Configuration

### Single Branch Strategy

The project now uses a single `main` branch for both Homebrew and App Store distribution. The auto-updater feature is **enabled by default** and conditionally disabled with the `DISABLE_AUTO_UPDATE` flag:

- **Default builds (Debug/Release)**: Auto-updater enabled
- **GitHub Actions/Homebrew**: Auto-updater enabled (default)
- **App Store builds**: Auto-updater disabled (use Debug-AppStore or add DISABLE_AUTO_UPDATE flag)

This approach is App Store compliant as disabled code paths are acceptable for review.

## Development Commands

### Build and Run
```bash
# Build for Development/Homebrew (auto-updater enabled by default)
xcodebuild -scheme ZenDevToolkit -configuration Debug

# Build for App Store (auto-updater disabled)
xcodebuild -scheme ZenDevToolkit -configuration Debug \
  SWIFT_ACTIVE_COMPILATION_CONDITIONS="DISABLE_AUTO_UPDATE"

# Build for release (Homebrew - auto-updater enabled by default)
xcodebuild -scheme ZenDevToolkit -configuration Release

# Build for release (App Store - auto-updater disabled)
xcodebuild -scheme ZenDevToolkit -configuration Release \
  SWIFT_ACTIVE_COMPILATION_CONDITIONS="DISABLE_AUTO_UPDATE"

# Clean build folder
xcodebuild -scheme ZenDevToolkit clean
```

### Signed Release Build
```bash
# Build, sign, and prepare for distribution
./scripts/build-release.sh

# This script:
# - Creates a signed archive
# - Exports with Developer ID certificate
# - Verifies code signature
# - Creates ZIP for notarization
# - Optionally submits for notarization (if credentials are set)
```

### Code Signing & Notarization

The app is configured for trusted distribution with:
- **Team ID**: 3Z86BP8YAG (Dileesha Rajapakse)
- **Bundle ID**: com.luminaxa.ZenDevToolkit
- **Code Signing**: Automatic with Developer ID Application certificate
- **Hardened Runtime**: Enabled
- **Sandboxing**: Enabled with file access permissions

#### Manual Notarization
```bash
# Set credentials (required for notarization)
export APPLE_ID="dilee.dev@gmail.com"
export APPLE_APP_PASSWORD="your-app-specific-password"

# Notarize the built app
./scripts/notarize.sh build/export-distribution/ZenDevToolkit.app

# Or notarize a ZIP file
./scripts/notarize.sh build/ZenDevToolkit.zip
```

#### Creating App-Specific Password
1. Go to https://appleid.apple.com/ (sign in with dilee.dev@gmail.com)
2. Navigate to Security section
3. Generate app-specific password for "ZenDevToolkit Notarization"
4. Use this password as `APPLE_APP_PASSWORD`

### Testing
```bash
# Run all tests
xcodebuild test -scheme ZenDevToolkit -destination 'platform=macOS'

# Run specific test target
xcodebuild test -scheme ZenDevToolkit -only-testing:ZenDevToolkitTests
```

### Open in Xcode
```bash
open ZenDevToolkit.xcodeproj
```

## Planned Features (v1.0)

1. **JSON Formatter & Validator** ✅ (Implemented)
   - Format, minify, validate with error messages
   - Clipboard integration, monospace font display

2. **Base64 Encoder/Decoder** (Placeholder)
   - Text and file support planned
   - Bidirectional encoding/decoding

3. **URL Encoder/Decoder** (Placeholder)
   - Percent-encoding for URLs
   - Query parameter handling

4. **Hash Generator** (Placeholder)
   - MD5, SHA1, SHA256 support planned
   - Uses CommonCrypto framework

5. **UUID Generator** (Placeholder)
   - Version 4 UUIDs
   - Multiple format options

6. **Timestamp Converter** ✅ (Implemented)
   - Convert Unix timestamps to human-readable dates
   - Convert human dates to Unix timestamps
   - Support for multiple timezones
   - Multiple date format support
   - Relative time display ("2 hours ago")

7. **JWT Token Tool** ✅ (Implemented)
   - Decode JWT tokens with header, payload, and signature display
   - Generate JWT tokens with custom claims

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dilee/zen-dev-toolkit](https://github.com/dilee/zen-dev-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
