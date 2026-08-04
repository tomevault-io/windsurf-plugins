---
trigger: always_on
description: <!-- GSD:project-start source:PROJECT.md -->
---

<!-- GSD:project-start source:PROJECT.md -->
## Project

**Goose — Self-Hosted Biometric Platform**

iOS app (SwiftUI + Rust core) that reads biometric data from WHOOP devices via BLE and persists it to a self-hosted server.

**Core Value:** Users must be able to capture WHOOP data on iPhone and have it automatically persisted on their personal server — without depending on external infrastructure.

### Constraints

- **iOS stack**: Swift / SwiftUI / URLSession — no external dependencies
- **Server stack**: FastAPI + TimescaleDB (Docker, self-hosted)
- **Git**: planning docs committed (commit_docs: true)
<!-- GSD:project-end -->

<!-- GSD:stack-start source:codebase/STACK.md -->
## Technology Stack

## Languages
- Swift 5.0 — iOS app, all UI and business logic in `GooseSwift/`, live activity extension in `GooseWorkoutLiveActivityExtension/`
- Rust (Edition 2024, MSRV 1.96) — Rust core library in `Rust/core/src/`, protocol parsing, metric computation, SQLite persistence, FFI bridge
- Python — Reference algorithm scripts only (`Rust/core/tools/reference/*.py`); not used at runtime
- Bash — Rust cross-compilation script at `Scripts/build_ios_rust.sh`
## Runtime
- iOS 26.0 (deployment target — `IPHONEOS_DEPLOYMENT_TARGET = 26.0` in `GooseSwift.xcodeproj/project.pbxproj`)
- ARM64 device (`aarch64-apple-ios`), ARM64 simulator (`aarch64-apple-ios-sim`), x86_64 simulator (`x86_64-apple-ios`) all supported via build script
- Swift: No SPM (no root `Package.swift`). Project managed via `GooseSwift.xcodeproj`. Two local packages exist in `Packages/WhoopProtocol/` and `Packages/WhoopStore/` but contain only `.swiftpm` metadata — no source files; they appear to be placeholder or removed packages.
- Rust: Cargo, lockfile at `Rust/core/Cargo.lock` (present, committed)
## Frameworks
- SwiftUI — all UI; 80 files import SwiftUI
- UIKit — used for appearance configuration and low-level UI hooks; 81 files import UIKit
- Foundation — universal; 97 files import Foundation
- CoreBluetooth — BLE communication with WHOOP device; 14 files import CoreBluetooth
- HealthKit — body mass autofill from Apple Health; 11 files import HealthKit; entitlement `com.apple.developer.healthkit` granted in `GooseSwift/GooseSwift.entitlements`
- CoreLocation + MapKit — outdoor workout GPS tracking; 12 files import CoreLocation, 9 import MapKit
- ActivityKit — Live Activity / Dynamic Island for workouts; `GooseSwift/WorkoutLiveActivityController.swift`, `GooseWorkoutLiveActivityExtension/GooseWorkoutLiveActivityWidget.swift`
- WidgetKit — Live Activity widget extension; `GooseWorkoutLiveActivityExtension/GooseWorkoutLiveActivityWidget.swift`
- OSLog — structured logging; 11 files import OSLog
- CryptoKit — SHA-256 file integrity checksums for export; 5 files import CryptoKit
- Security — iOS Keychain for OAuth token storage; `GooseSwift/CodexEmbeddedAuth.swift`
- UserNotifications — notification permission onboarding; `GooseSwift/OnboardingModels.swift`, `GooseSwift/OnboardingPermissions.swift`
- Rust: Cargo's built-in test runner (`cargo test`). Integration tests in `Rust/core/tests/` (47 files). Swift tests in `GooseSwiftTests/` target (69 tests across 16 files).
- Xcode project: `GooseSwift.xcodeproj`
- Rust cross-compile: `Scripts/build_ios_rust.sh` — invoked as an Xcode build phase, produces `Rust/iphoneos/libgoose_core.a` and `Rust/iphonesimulator/libgoose_core.a`
- Python reference tools: `Rust/core/tools/reference/` — neurokit2, pyhrv, pyactigraphy, ggir; used only for algorithm validation/comparison, not production
## Key Dependencies
- `rusqlite 0.37` (feature: `bundled`) — SQLite embedded in the static library; all health/capture/activity persistence goes through this
- `serde 1.0` + `serde_json 1.0` — all JSON serialisation for the FFI bridge protocol
- `tungstenite 0.28` — WebSocket server used for local debug sessions (`ws://127.0.0.1:8765`)
- `zip 0.6` — raw data export bundling
- `sha2 0.10` — SHA-256 digests inside Rust (separate from Swift CryptoKit usage)
- `crc32fast 1.4` — CRC32 frame checksums
- `hex 0.4` — hex encoding for BLE frame capture
- `thiserror 2.0` — error type derivation
- `tempfile 3.13` (dev-only) — test temporary files
## Configuration
- No `.env` files. Configuration driven by `ProcessInfo.processInfo` launch arguments and environment variables at runtime:
- `GooseSwift.xcodeproj` — main Xcode project
- `Scripts/build_ios_rust.sh` — Rust cross-compilation invoked as Xcode build phase; reads `PLATFORM_NAME`, `CONFIGURATION`, `CURRENT_ARCH`, `IPHONEOS_DEPLOYMENT_TARGET` from Xcode environment
- Bundle ID: `com.goose.app` (main app), `com.goose.app.WorkoutLiveActivityExtension` (extension)
- Marketing version: `8.0`, build: `8`
- URL scheme: `gooseapp://` (`CFBundleURLSchemes` in `GooseSwift/Info.plist`)
## Platform Requirements
- macOS with Xcode (iOS 26.0 SDK)
- Rust toolchain with targets: `aarch64-apple-ios`, `aarch64-apple-ios-sim`, `x86_64-apple-ios`
- Cargo (installed separately or via rustup)
- iOS device or simulator, iOS 26.0+
- Static libraries `Rust/iphoneos/libgoose_core.a` and `Rust/iphonesimulator/libgoose_core.a` are **gitignored** — built automatically by Xcode via `Scripts/build_ios_rust.sh` build phase
- Bluetooth background mode required (`UIBackgroundModes: bluetooth-central`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tigercraft4/goose](https://github.com/tigercraft4/goose) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
