---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SundialKitCombine is a Combine-based observation plugin for SundialKit that provides reactive network monitoring and WatchConnectivity communication for SwiftUI applications. It uses `@Published` properties and Combine publishers to deliver state updates, with strict `@MainActor` isolation for thread safety.

**Key characteristics:**
- Swift Package Manager library (no executables)
- Depends on SundialKit (main package with core protocols)
- Swift 6.1+ with strict concurrency enabled
- Platform support: iOS 16+, watchOS 9+, tvOS 16+, macOS 11+
- Targets Combine-based SwiftUI apps requiring backward compatibility

## Development Commands

### Building and Testing

```bash
# Build the package
swift build

# Build including tests
swift build --build-tests

# Run all tests
swift test

# Run tests with parallel execution
swift test --parallel

# Run a specific test
swift test --filter SundialKitCombineTests
```

### Code Quality

```bash
# Run all linting (format + lint + build)
LINT_MODE=STRICT ./Scripts/lint.sh

# Format only (no linting or build)
FORMAT_ONLY=1 ./Scripts/lint.sh

# Run linting in normal mode (without strict mode)
./Scripts/lint.sh

# Run periphery to detect unused code (requires mise)
mise exec -- periphery scan --disable-update-check
```

**Note:** Linting requires `mise` to be installed. The script checks for mise at:
- `/opt/homebrew/bin/mise`
- `/usr/local/bin/mise`
- `$HOME/.local/bin/mise`

Tools managed by mise (defined in `.mise.toml`):
- `swift-format` (version 602.0.0)
- `swiftlint` (version 0.61.0)
- `periphery` (version 3.2.0)

### Documentation

```bash
# Preview documentation locally
./Scripts/preview-docs.sh
```

## Architecture

### Three-Layer Architecture

SundialKitCombine sits in Layer 2 of the SundialKit architecture:

**Layer 1: Core Protocols** (dependencies from SundialKit package)
- `SundialKitCore`: Base protocols, errors, and types
- `SundialKitNetwork`: Network monitoring protocols (`PathMonitor`, `NetworkPing`)
- `SundialKitConnectivity`: WatchConnectivity protocols and messaging

**Layer 2: Observation Plugin** (this package)
- `NetworkObserver`: Combines `@MainActor` isolation with `@Published` properties for network state
- `ConnectivityObserver`: Manages WatchConnectivity session with Combine publishers

### Key Components

**NetworkObserver** (`Sources/SundialKitCombine/NetworkObserver.swift`)
- Generic over `MonitorType: PathMonitor` and `PingType: NetworkPing`
- `@MainActor` isolated with `@Published` properties:
  - `pathStatus`: Current network path status
  - `isExpensive`: Whether connection is cellular/expensive
  - `isConstrained`: Whether low data mode is active
  - `pingStatus`: Optional ping verification result
- Delegates to underlying monitor (typically `NWPathMonitor` via `NWPathMonitorAdapter`)
- Optional periodic ping for verification beyond path availability

**ConnectivityObserver** (`Sources/SundialKitCombine/ConnectivityObserver.swift`)
- `@MainActor` isolated observer for WatchConnectivity
- `@Published` properties for session state:
  - `activationState`: Session activation state
  - `isReachable`: Whether counterpart device is available
  - `isPairedAppInstalled`: Whether companion app is installed
  - `isPaired`: (iOS only) Whether Apple Watch is paired
  - `activationError`: Last activation error if any
- Combine `PassthroughSubject` publishers:
  - `messageReceived`: Raw messages with context
  - `typedMessageReceived`: Decoded `Messagable` types
  - `sendResult`: Message send confirmations
  - `activationCompleted`: Activation results with errors
- Supports `MessageDecoder` for type-safe message handling

**Message Types**
- `Messagable`: Protocol for dictionary-based type-safe messages (~65KB limit)
- `BinaryMessagable`: Protocol for efficient binary serialization (no size limit, works with Protobuf/custom formats)
- `MessageDecoder`: Decodes incoming messages to typed `Messagable` instances

### Concurrency Model

- All observers are `@MainActor` isolated
- No `@unchecked Sendable` conformances (strict concurrency compliance)
- Network/connectivity events marshaled to main actor via `Task { @MainActor in ... }`
- Natural integration with SwiftUI's `ObservableObject` pattern

## Swift Settings

The package enables extensive Swift 6.2 upcoming features and experimental features (see `Package.swift:8-45`):

**Upcoming Features:**
- `ExistentialAny`, `InternalImportsByDefault`, `MemberImportVisibility`, `FullTypedThrows`

**Experimental Features:**
- Noncopyable generics, move-only types, borrowing switch, init accessors, etc.

**Important:** When adding new code, maintain strict concurrency compliance. Avoid `@unchecked Sendable`.

## Code Style

- **Indentation:** 2 spaces (configured in `.swift-format`)
- **Line length:** 100 characters max
- **Documentation:** All public declarations must have documentation comments (`AllPublicDeclarationsHaveDocumentation` rule enabled)
- **ACL:** Explicit access control required on all declarations (`explicit_acl`, `explicit_top_level_acl`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brightdigit) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
