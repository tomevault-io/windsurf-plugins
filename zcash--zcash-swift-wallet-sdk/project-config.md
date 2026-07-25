---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

`ZcashLightClientKit` is an iOS/macOS Swift Package that implements a Zcash lightwallet client. The Swift layer wraps a Rust core (in `rust/`) via an `libzcashlc` XCFramework. Most day-to-day SDK work happens in Swift only — SPM auto-downloads a pre-built XCFramework from GitHub Releases.

## Build and test

Open the package or workspace in Xcode and build against an iOS or macOS target:

- `swift build` — build the package (macOS target).
- `swift test --filter OfflineTests` — run the offline unit tests. This is what CI runs (see `.github/workflows/swift.yml`).
- `xcodebuild ... -testPlan ZcashLightClientKit.xctestplan` — the shared test plan enables only `OfflineTests`; other test targets are disabled by default and must be enabled manually when needed.

Test targets are grouped by external dependencies:

| Target | Requires |
|---|---|
| `OfflineTests` | nothing |
| `NetworkTests` | internet connection |
| `DarksideTests` / `AliasDarksideTests` | a local `lightwalletd` (`Tests/lightwalletd/lightwalletd --no-tls-very-insecure --data-dir /tmp --darkside-very-insecure --log-file /dev/stdout`); optionally set `LIGHTWALLETD_ADDRESS` |
| `PerformanceTests` | network, not run in CI |

## Rust FFI development

The Rust code in `rust/` is compiled into the `libzcashlc` XCFramework. Two modes, switched automatically by `Package.swift` based on whether `LocalPackages/Package.swift` exists:

- **Binary release mode** (default): `.binaryTarget` in `Package.swift` pulls the XCFramework zip from the GitHub Release referenced there (URL + checksum).
- **Local FFI mode**: `LocalPackages/` acts as a path-dependency override. The workspace's `FFIBuilder` target auto-rebuilds on Xcode builds.

Scripts:

- `./Scripts/init-local-ffi.sh` — one-time setup; default builds all 5 architectures and creates `LocalPackages/`. Arm-only subsets (faster on Apple Silicon — they skip the x86_64 slices): **`--arm-macos`** (macOS slice only, good for `swift build` / `swift test` on the Mac), **`--arm-ios`** (iOS simulator + device slices), **`--arm-all`** (iOS simulator + device + macOS). Use `--cached` only when your branch has no FFI changes relative to the release.
- `./Scripts/rebuild-local-ffi.sh [ios-sim|ios-device|macos]` — fast single-arch incremental rebuild after Rust edits. `ios-sim` is default.
- `./Scripts/reset-local-ffi.sh` — remove `LocalPackages/` and switch back to the release binary.

For FFI work, open `ZcashSDK.xcworkspace` (not `Package.swift`) so `FFIBuilder` auto-runs. After switching modes or if headers look stale, in Xcode: Cmd+Shift+K, then File > Packages > Reset Package Caches. When modifying the Rust/Swift FFI boundary, run the full `init-local-ffi.sh` before PRing — `rebuild-local-ffi.sh` only covers one arch.

See `docs/LOCAL_DEVELOPMENT.md` for the full reference.

## Release

- `./Scripts/release.sh <remote> <version>` — fully automated release (bumps the XCFramework URL+checksum in `Package.swift`, signs a tag, drafts GitHub Release).
- `./Scripts/prepare-release.sh <version>` — semi-automated alternative.
- The `Build FFI XCFramework` GitHub Action (`workflow_dispatch`) produces release artifacts.

## Architecture

### Two-layer wallet

1. **Rust core** (`rust/src/`) — key derivation, note scanning, transaction construction, block database migrations.
2. **Swift SDK** (`Sources/ZcashLightClientKit/`) — orchestration, networking, persistence, public API.

The Swift↔Rust bridge lives in `Sources/ZcashLightClientKit/Rust/`:
- `ZcashRustBackend` conforms to `ZcashRustBackendWelding` — the DB-bound surface.
- `ZcashKeyDerivationBackend` conforms to `ZcashKeyDerivationBackendWelding` — the stateless key-derivation surface.

Both are the only callers of the generated C header `libzcashlc`.

### Synchronizer is the public entry point

- `Synchronizer.swift` defines the public protocol.
- `SDKSynchronizer` (in `Synchronizer/SDKSynchronizer.swift`) is the concrete actor-based implementation. `ClosureSDKSynchronizer` and `CombineSDKSynchronizer` (plus the `ClosureSynchronizer`/`CombineSynchronizer` top-level files) are thin adapters over the `async/await` API. Prefer extending the async API and letting the adapters delegate.
- `Synchronizer/Dependencies.swift` is the DI composition root — it wires the entire object graph (repositories, services, rust backend, compact block processor, Tor client). Most "where does X come from?" questions are answered here.
- `Initializer.swift` is the user-facing entry point that validates paths, configures logging, and hands config to `Synchronizer`.

### Sync pipeline: CompactBlockProcessor + Actions

`Block/CompactBlockProcessor.swift` is a Swift actor that drives a state machine (`CBPState`) over an ordered list of `Block/Actions/*Action.swift` units: download → validate server → update chain tip → update subtree roots → process suggested scan ranges → scan → enhance → fetch UTXOs → clear cache → resubmit / migrate legacy / rewind. Each `Action` conforms to the protocol in `Block/Actions/Action.swift` and mutates a shared `ActionContext`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zcash/zcash-swift-wallet-sdk](https://github.com/zcash/zcash-swift-wallet-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
