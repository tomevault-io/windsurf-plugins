---
trigger: always_on
description: Guidelines for AI coding agents working in this repository. This is a Swift framework
---

# AGENTS.md — PeerConnectivity

Guidelines for AI coding agents working in this repository. This is a Swift framework
wrapping Apple's MultipeerConnectivity for Bluetooth/WiFi mesh networking.

## Build & Test Commands

### Build (Xcode — primary)

```bash
# Build the framework (iOS Simulator)
xcodebuild -workspace PeerConnectivity.xcworkspace \
  -scheme PeerConnectivity \
  -destination 'platform=iOS Simulator,name=iPhone 16' \
  -configuration Debug build

# Build via Swift Package Manager (no test target in Package.swift)
swift build
```

### Run Tests

```bash
# Run all tests
xcodebuild test -workspace PeerConnectivity.xcworkspace \
  -scheme PeerConnectivity \
  -destination 'platform=iOS Simulator,name=iPhone 16' \
  -configuration Debug

# Run a single test class
xcodebuild test -workspace PeerConnectivity.xcworkspace \
  -scheme PeerConnectivity \
  -destination 'platform=iOS Simulator,name=iPhone 16' \
  -only-testing:PeerConnectivityTests/PeerMessageTests

# Run a single test method
xcodebuild test -workspace PeerConnectivity.xcworkspace \
  -scheme PeerConnectivity \
  -destination 'platform=iOS Simulator,name=iPhone 16' \
  -only-testing:PeerConnectivityTests/PeerMessageTests/testSimpleMessageRoundTrip
```

### No Linter Configured

No SwiftLint or SwiftFormat is set up. Follow the style conventions below.

## Project Layout

```
Sources/                    # All library source (single SPM target)
PeerConnectivityTests/      # XCTest unit tests (Xcode-only target)
PeerConnectivityDemo/       # Demo iOS app
PeerPlayground.playground/  # Interactive examples
Package.swift               # SPM manifest (iOS 8+, macOS 10.10+, Swift 5)
PeerConnectivity.podspec    # CocoaPods spec (v0.5.4)
```

Key files: `PeerConnectionManager.swift` (630 lines, main public API),
`PeerConnectionResponder.swift` (event dispatch), `Peer.swift` (peer model).

## Code Style

### Imports

Order: `Foundation` first, then `MultipeerConnectivity`, then platform imports
wrapped in `#if os(iOS)` / `#elseif os(macOS)`. No blank lines between imports.
Zero third-party dependencies.

```swift
import Foundation
import MultipeerConnectivity
#if os(iOS)
import UIKit
#elseif os(macOS)
import AppKit
#endif
```

### Formatting

- **Indentation**: 4 spaces (no tabs)
- **Braces**: K&R / same-line opening brace (Swift standard)
- **Line length**: Soft limit ~120 chars; delegate signatures may be longer
- **Blank lines**: Single blank line between logical sections; double blank line
  occasionally separates major property groups
- **Space before colon in type annotations**: This codebase uses `let foo : Type`
  (non-standard but consistent — follow it)
- **Dictionary types**: Prefer `[String:Any]` (no spaces around colon in dict types)
- **Trailing commas**: Used in multi-line lists

### Access Control

Always explicit — write `internal` even though it's the default.

| Level | Usage |
|-------|-------|
| `public` | External API: `PeerConnectionManager`, `Peer`, `PeerConnectionEvent`, protocols, typealiases |
| `internal` | All wrapper types, observables, event producers, internal event enums |
| `fileprivate` | Stored properties of wrapper types, observer properties |
| `private` | Rare; test helpers only |

Special patterns used: `public fileprivate(set)` for read-only public properties,
`internal fileprivate(set)` for responder listeners.

### Naming Conventions

- **Types**: PascalCase — `PeerConnectionManager`, `PeerSessionEvent`
- **Related type naming pattern**: `Peer*` (wrapper), `Peer*EventProducer` (delegate bridge), `Peer*Event` (event enum)
- **Variables/properties**: camelCase — `connectionType`, `foundPeers`, `sessionObserver`
- **Functions**: camelCase with argument labels — `sendData(_:toPeers:)`, `listenOn(_:performListenerInBackground:withKey:)`
- **Enum cases**: camelCase with labeled associated values — `.devicesChanged(peer:connectedPeers:)`
- **Typealiases**: PascalCase — `ServiceType`, `PeerConnectionEventListener`
- **Static constants**: PascalCase — `PeerConnectivityKeys.CertificateListener`
- **IMPORTANT misspelling**: The codebase uses "Assisstant" (double 's') and "Dissmiss" intentionally. Maintain these in existing type/method names for API compatibility.

### Documentation

- **File headers**: Every file has the standard Xcode header (filename, project, author, copyright)
- **Public APIs**: Use `/** */` multi-line doc comments with `- parameter name:` and `- Returns:` format
- **Enum cases**: Use `/** */` doc comments
- **Internal helpers**: Use `///` single-line doc comments
- **Section markers**: `// MARK:` and `// MARK: -` to organize code within files
- **Suppressed docs**: Use `/// :nodoc:` for boilerplate conformances (Hashable, Equatable)

### Error Handling

```swift
// Pattern 1 (most common): do/catch with NSLog
do {
    try session.send(data, toPeers: peers, with: .reliable)
} catch let error {
    NSLog("%@", "Error sending data: \(error)")
}

// Pattern 2: try? for optional/silent failure
guard let data = try? NSKeyedArchiver.archivedData(...) else { return }

// Pattern 3: rethrow
do { return try session.sendDataStream(name, toPeer: peer) }
catch let error { throw error }
```

No custom Error types — uses system errors. Logging via `NSLog("%@", ...)`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rchatham/PeerConnectivity](https://github.com/rchatham/PeerConnectivity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
