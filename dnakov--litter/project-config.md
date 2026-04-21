---
trigger: always_on
description: - `apps/ios/Sources/Litter/` contains the iOS app code.
---

# Repository Guidelines

## Project Structure & Module Organization
- `apps/ios/Sources/Litter/` contains the iOS app code.
- `apps/ios/Sources/Litter/Views/` holds SwiftUI screens, `Models/` contains app state/session logic, and `Bridge/` contains JSON-RPC + C FFI bridge code.
- `apps/android/app/src/main/java/com/litter/android/ui/` contains Android Compose shell/screens.
- `apps/android/app/src/main/java/com/litter/android/state/` contains Android app state, server/session manager, SSH, and websocket transport.
- `apps/android/core/bridge/` contains Android UniFFI bootstrap and generated Rust bindings.
- `apps/android/app/src/test/java/` contains Android unit tests.
- `apps/android/docs/qa-matrix.md` tracks Android parity QA coverage.
- `shared/rust-bridge/codex-mobile-client/` is the single shared Rust client library consumed by both iOS and Android. It owns the public UniFFI surface, generated upstream RPC coverage, canonical store/reducer state, hydration, discovery, SSH, and shared runtime logic. `MobileClient` is the top-level internal Rust facade.
- `shared/rust-bridge/codex-ios-audio/` contains the iOS-only audio/AEC implementation used by `codex-mobile-client`.
- `shared/rust-bridge/codex-bridge/` is legacy C-FFI support that should not be used for new mobile runtime features.
- `apps/ios/Sources/Litter/Bridge/Rust*.swift` — iOS bridge files mapping Swift to the shared Rust layer.
- `apps/android/core/bridge/.../Rust*.kt` — Android bridge files mapping Kotlin to the shared Rust layer. UniFFI Kotlin sources are generated into `shared/rust-bridge/generated/kotlin/` and consumed directly from there; do not maintain copied binding files under Android source roots.
- `shared/third_party/codex/` is the upstream Codex submodule.
- `apps/ios/GeneratedRust/` contains local generated Rust artifacts for iOS builds: UniFFI headers/modulemap plus raw device/simulator staticlibs. These artifacts are not committed.
- `apps/ios/Frameworks/` contains downloaded/package-lane iOS XCFrameworks (`codex_mobile_client.xcframework` in package builds and `ios_system/*`). These artifacts are not committed.
- `apps/ios/project.yml` is the source of truth for project generation; regenerate `apps/ios/Litter.xcodeproj` instead of hand-editing project files.

## Architecture
- **iOS root layout:** `ContentView` uses a `ZStack` with a persistent `HeaderView`, main content area, and a `SidebarOverlay` that slides from the left.
- **iOS state management:** `AppStore` (Rust, via UniFFI) is the canonical runtime state owner. `AppModel` is the thin Swift observation shell over Rust snapshots and updates. `AppState` is UI-only state.
- **iOS server flow:** discovery and SSH are separate utility bridges; thread/session/account operations come from generated Rust RPC plus store updates.
- **Android root layout:** `LitterAppShell` is the Compose entry; `DefaultLitterAppState` maps backend state into UI state.
- **Android state/transport:** Android should use the same Rust-owned runtime model as iOS instead of re-implementing shared session/thread/account logic in Kotlin.
- **Android server flow:** discovery seeds come from Android NSD, but discovery merge/probe policy lives in Rust; connection, auth, and thread/account flows go through Rust RPC + store updates.
- **Message rendering parity:** both platforms support reasoning/system sections, code block rendering, and inline image handling.

### Shared Rust Layer
- `codex-mobile-client` is the single public Rust mobile crate. Keep one generated Swift/Kotlin binding surface; do not split UniFFI across multiple mobile crates again.
- `codex-ios-audio` is the separate iOS-only audio/AEC crate. Keep heavy audio processing there, not in Swift and not in a second UniFFI crate.
- `AppStore` is the Rust-owned state surface. It owns snapshots, typed updates, and the small set of truly composite/store-local actions.
- `AppClient` is the public UniFFI client surface for direct server operations and typed results.
- `DiscoveryBridge` and `SshBridge` are separate Rust utility surfaces. Do not move discovery/SSH policy back into Swift/Kotlin.
- iOS uses UniFFI-generated Swift plus thin bridge helpers; Android uses UniFFI-generated Kotlin plus thin bridge helpers.
- iOS Debug/device links the raw static library in `apps/ios/GeneratedRust/ios-device/libcodex_mobile_client.a`. Package/release lanes may still create `apps/ios/Frameworks/codex_mobile_client.xcframework`, but that is not the default debug/device artifact.

## Feature Placement Rules
- Prefer Rust first. If logic is about session state, thread state, streaming, hydration, approvals, auth/account, discovery merge policy, voice transcript/handoff normalization, or status normalization, it belongs in `shared/rust-bridge/codex-mobile-client/`.
- Keep Swift/Kotlin thin. Platform code should only own UI, platform persistence, platform permissions, audio/session APIs, notifications, ActivityKit/CarPlay/Android services, and render-only projections.
- Do not parse upstream wire-format strings in Swift/Kotlin. If a status, event kind, or payload shape matters to both platforms, expose it as a typed UniFFI enum/record from Rust.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dnakov/litter](https://github.com/dnakov/litter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
