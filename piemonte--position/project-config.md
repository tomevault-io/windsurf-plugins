---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## ⚠️ CRITICAL: Never Use Force Unwrap (!)

**Force unwrapping with `!` is prohibited in this codebase. ALWAYS use safe optional handling:**

### ❌ NEVER Do This:
```swift
let location = locationManager.location!  // Crash if nil
let coordinate = location.coordinate!     // Crash if nil
```

### ✅ ALWAYS Do This Instead:

**Use guard let for early returns:**
```swift
guard let location = locationManager.location else {
    print("Location unavailable")
    return
}
// Safe to use location here
```

**Use optional chaining:**
```swift
if let coordinate = location?.coordinate {
    // Use coordinate safely
}
```

**Use nil coalescing for defaults:**
```swift
let accuracy = location?.horizontalAccuracy ?? 0.0
```

**Why this matters:** Force unwraps cause immediate crashes when encountering nil values. Safe optional handling ensures the library remains stable and provides better error handling for consumers.

## Project Overview

Position is a Swift 6-ready, actor-based location positioning library for iOS and macOS with modern async/await APIs and AsyncSequence support. The library provides thread-safe location services with Swift concurrency.

## Requirements

- iOS 16.0+ / macOS 13.0+
- Swift 6.0+ (also supports Swift 5 mode)
- Xcode 16.0+

## Commands

### Build Commands
```bash
# Build for debug
swift build

# Build for release
swift build -c release

# Run tests
swift test

# Clean build
swift package clean
```

### Linting (SwiftLint must be installed)
```bash
# Lint sources
swiftlint lint Sources/

# Auto-fix linting issues
swiftlint --fix Sources/
```

### Build with Xcode
```bash
# iOS Simulator
xcodebuild -scheme Position -destination 'platform=iOS Simulator,name=iPhone 15'

# macOS
xcodebuild -scheme Position -destination 'platform=macOS'
```

## Architecture

### Core Components

1. **Position.swift** (Sources/Position.swift:77-900+)
   - Main actor-based class that manages all location services
   - Provides async/await APIs for one-shot location requests
   - Implements AsyncSequence streams for continuous updates (location, heading, authorization, floor, visit, errors)
   - Maintains backwards compatibility with deprecated observer pattern

2. **DeviceLocationManager.swift** (Sources/DeviceLocationManager.swift)
   - Internal actor that wraps CLLocationManager
   - Handles platform-specific location services
   - Manages delegation from CoreLocation to Position actor
   - Implements battery and background state monitoring

3. **CoreLocation+Additions.swift** (Sources/CoreLocation+Additions.swift)
   - Extensions for CLLocation, CLPlacemark, CLLocationCoordinate2D
   - Geospatial calculation utilities (distance, bearing, coordinate at bearing)
   - vCard generation for location sharing
   - Pretty formatting methods for locations and addresses

### Swift 6 Concurrency Model

Position follows strict Swift 6 concurrency patterns:

- **Actor Isolation**: Position is an actor, ensuring thread-safe access to all location services
- **Async/Await**: All public methods are async and must be called with await
- **AsyncSequence**: Reactive streams for continuous updates without callbacks
- **@preconcurrency**: Used for CoreLocation imports to handle API evolution
- **Sendable**: All data types passed across actors conform to Sendable

**AsyncSequence Streams:**
- `locationUpdates`: Location changes
- `headingUpdates`: Compass heading (iOS only)
- `authorizationUpdates`: Permission status changes
- `floorUpdates`: Indoor floor level changes
- `visitUpdates`: Significant location visits
- `errorUpdates`: Location errors

### Platform Support

- iOS 16.0+ / macOS 13.0+
- Swift 6.0+ (also supports Swift 5 mode)
- Xcode 16.0+
- Platform-specific features:
  - Heading updates: iOS only
  - Battery monitoring: iOS only
  - Background updates: Full on iOS, limited on macOS

## Swift 6 Concurrency Best Practices

### Actor Usage

**Position as an actor:**
```swift
public actor Position {
    // All methods are actor-isolated
    public func currentLocation() async throws -> CLLocation {
        // Thread-safe by default
    }
}
```

**Calling actor methods:**
```swift
let position = Position()
let location = try await position.currentLocation()  // Must use await
```

### Async/Await Patterns

**One-shot location request:**
```swift
do {
    let location = try await position.currentLocation()
    print("Location: \(location.coordinate)")
} catch {
    print("Error: \(error)")
}
```

**AsyncSequence for continuous updates:**
```swift
Task {
    for await location in position.locationUpdates {
        print("New location: \(location.coordinate)")
    }
}
```

**Authorization handling:**
```swift
let status = try await position.requestWhenInUseLocationAuthorization()
switch status {
case .allowedWhenInUse:
    print("Authorized")
case .denied:
    print("Access denied")
default:
    print("Other status: \(status)")
}
```

### Task Cancellation Pattern

**CRITICAL: All long-running tasks MUST be cancellable to prevent memory leaks.**

```swift
class LocationViewModel {
    private var locationTask: Task<Void, Never>?

    deinit {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [piemonte/Position](https://github.com/piemonte/Position) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
