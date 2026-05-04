---
trigger: always_on
description: **Celluloid** is a macOS virtual camera application that captures video from physical cameras, applies real-time filters and adjustments, and outputs processed video as a virtual camera for use in video conferencing apps (Zoom, Google Meet, FaceTime, etc.).
---

# Celluloid - GitHub Copilot Instructions

## Project Overview

**Celluloid** is a macOS virtual camera application that captures video from physical cameras, applies real-time filters and adjustments, and outputs processed video as a virtual camera for use in video conferencing apps (Zoom, Google Meet, FaceTime, etc.).

**Key Facts:**
- **Language:** Swift 5.0
- **Frameworks:** SwiftUI, AVFoundation, CoreMediaIO (CMIOExtension), CoreImage
- **Deployment Target:** macOS 26.1+ (macOS 15 Sequoia)
- **Project Size:** ~2,500 lines of Swift code across 9 source files
- **Architecture:** Two-component system (Main App + System Extension)
- **Build System:** Xcode 26.1.1, no external build tools (no CocoaPods, SPM dependencies, or Carthage)
- **Code Signing:** Requires Team ID `36ERVRQ23S` for both app and extension

## Architecture

The app consists of two main components running in separate processes:

### 1. Main App (`Celluloid/` directory)
A SwiftUI menu bar application that:
- Captures video from physical cameras using AVFoundation
- Applies real-time image processing (brightness, contrast, saturation, exposure, temperature, filters, LUTs)
- Sends processed frames to the camera extension via CMIOExtension sink stream
- Manages the system extension lifecycle (install/update)

**Key Files:**
- `CelluloidApp.swift` (35 lines) - App entry point, menu bar setup
- `ContentView.swift` (449 lines) - Main UI with camera preview, adjustments, filter controls
- `CameraManager.swift` (1,202 lines) - **Core logic**: AVFoundation capture, filter processing, sink stream connection
- `ExtensionManager.swift` (125 lines) - SystemExtensions framework integration

### 2. Camera Extension (`CelluloidCameraExtension/` directory)
A CMIOExtension (CoreMediaIO System Extension) that:
- Registers as a virtual camera device named "Celluloid Camera" in macOS
- Receives processed frames from the main app via sink stream
- Provides video stream to any app requesting camera input

**Key Files:**
- `main.swift` (14 lines) - Extension entry point
- `CelluloidProviderSource.swift` (57 lines) - CMIOExtension provider
- `CelluloidDeviceSource.swift` (93 lines) - Device provider, connects sink and source streams
- `CelluloidSinkStream.swift` (203 lines) - Receives frames from main app
- `CelluloidStreamSource.swift` (314 lines) - Outputs frames to video apps

## Building and Testing

### Prerequisites
- **macOS 15.0 (Sequoia) or later** - This project ONLY builds on macOS
- **Xcode 26.1.1** or later
- Apple Developer account with valid Team ID for code signing

### Building the App

**⚠️ IMPORTANT:** The build process automatically installs the app to `/Applications/`. This is **REQUIRED** for system extensions to work. Running the app from Xcode's build directory will fail.

#### Build Commands

```bash
# Open project in Xcode
open Celluloid.xcodeproj

# Build from command line (Debug configuration)
xcodebuild -project Celluloid.xcodeproj -scheme Celluloid -configuration Debug build

# Build from command line (Release configuration)
xcodebuild -project Celluloid.xcodeproj -scheme Celluloid -configuration Release build

# Build and archive for distribution
xcodebuild -project Celluloid.xcodeproj -scheme Celluloid -configuration Release archive -archivePath ./build/Celluloid.xcarchive
```

**Build Output Locations:**
- Debug builds: `~/Library/Developer/Xcode/DerivedData/Celluloid-*/Build/Products/Debug/Celluloid.app`
- Release builds: `~/Library/Developer/Xcode/DerivedData/Celluloid-*/Build/Products/Release/Celluloid.app`

**Build Time:** Typical clean build takes 30-60 seconds on modern Mac hardware.

### Running the App

1. **Build the app** using Xcode (⌘+B) or xcodebuild
2. **Copy to /Applications/**: The app MUST be in `/Applications/` for system extensions to work
3. **Launch** the app from `/Applications/Celluloid.app`
4. **Approve system extension** when prompted (first launch only)
5. The virtual camera "Celluloid Camera" will appear in video apps after approval

**Note:** The app runs as a menu bar app (LSUIElement = true), so it won't appear in the Dock.

### Testing

The project uses Swift Testing framework (not XCTest).

**Test Files:**
- `CelluloidTests/CelluloidTests.swift` (482 lines)
- Tests cover: FilterType enum, settings boundaries, LUT parsers (Cube and HALD CLUT), adjustment resets

**Running Tests:**

```bash
# Run all tests from Xcode
# Use Test Navigator (⌘+6) or Product > Test (⌘+U)

# Run tests from command line
xcodebuild test -project Celluloid.xcodeproj -scheme Celluloid -destination 'platform=macOS'

# Run specific test
xcodebuild test -project Celluloid.xcodeproj -scheme Celluloid -destination 'platform=macOS' -only-testing:CelluloidTests/CelluloidTests/filterTypeHasCorrectCases
```

**Test Execution Time:** Full test suite runs in ~2-5 seconds.

**UI Tests:** The `CelluloidUITests/` directory exists but is currently empty. UI testing is not yet implemented.

## Code Organization

### Directory Structure
```
Celluloid/
├── Celluloid/                          # Main app target

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whyisjake/Celluloid](https://github.com/whyisjake/Celluloid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
