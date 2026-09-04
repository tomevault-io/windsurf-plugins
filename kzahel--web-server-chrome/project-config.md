---
trigger: always_on
description: Read [docs/vision.md](docs/vision.md) first for product intent. For current
---

# 200 OK Web Server

Read [docs/vision.md](docs/vision.md) first for product intent. For current
architecture and implementation direction, read
[docs/topics/desktop-runtime.md](docs/topics/desktop-runtime.md),
[docs/topics/android-runtime.md](docs/topics/android-runtime.md),
[docs/topics/ios-runtime.md](docs/topics/ios-runtime.md), and
[docs/topics/chromeos-extension-launcher.md](docs/topics/chromeos-extension-launcher.md).
For the accepted Play-free Linux product shape and user flow, also read
[docs/topics/chromeos-crostini-launcher.md](docs/topics/chromeos-crostini-launcher.md).
For the active final release gate and the split between agent-owned and
maintainer/device checks, read
[docs/tactical/009-release-confidence-closeout.md](docs/tactical/009-release-confidence-closeout.md)
and
[docs/tactical/011-extension-launcher-and-chromeos-network-readiness.md](docs/tactical/011-extension-launcher-and-chromeos-network-readiness.md).
For the active desktop defect repair and the exact-public-artifact acceptance
campaign, read
[docs/tactical/015-desktop-production-validation.md](docs/tactical/015-desktop-production-validation.md)
and follow
[docs/runbooks/desktop-production-validation.md](docs/runbooks/desktop-production-validation.md).
The scoped Play-free ChromeOS Linux fallback lives in
[docs/tactical/012-chromeos-crostini-fallback.md](docs/tactical/012-chromeos-crostini-fallback.md).
The active ChromeOS Linux product-completion plan lives in
[docs/tactical/014-chromeos-crostini-product-completion.md](docs/tactical/014-chromeos-crostini-product-completion.md).
The native iOS MVP is complete in
[docs/tactical/016-native-swift-ios-app.md](docs/tactical/016-native-swift-ios-app.md);
its separate store-readiness follow-up is
[docs/tactical/017-ios-store-readiness.md](docs/tactical/017-ios-store-readiness.md).
Cross-platform CI, shared HTTP conformance, product E2E, artifact validation,
compatibility fixtures, and advisory testbed handoff are implemented in
[docs/tactical/018-cross-platform-ci-and-test-confidence.md](docs/tactical/018-cross-platform-ci-and-test-confidence.md).

## Quick Context

Lightweight web server app for every platform. Successor to "Web Server for
Chrome" (200k+ users). The Android app, extension, ChromeOS Linux component,
and signed Rust-native Tauri desktop `v0.1.6` have shipped.

Desktop `v0.1.6` passed its signing and public-asset gates but failed later
three-OS functional acceptance: the settings surface is clipped on Windows and
Linux, disabling background operation does not exit on last-window close, and
Windows can become unrecoverably invisible with the tray hidden. It also lacks
an exact-version round trip through the production Chrome Web Store extension.
Do not describe it as production-accepted; Tactical 015 owns the repair and
post-publication rerun.

A standalone native SwiftUI/Swift iOS app now exists and has passed its
simulator, Release-hygiene, Files/bookmark, lifecycle, and external same-Wi-Fi
acceptance gates on the attached physical phone through the project-neutral
`~/code/ios-device-testbed` path. It is not released; Tactical 017 owns the
separate App Store/TestFlight lane.

The old Transistor proof is not the current desktop architecture. Desktop
keeps Tauri and its webview for control/configuration while a small Rust core
owns HTTP execution on Windows, macOS, and Linux. Desktop `v0.1.6` adds the
canonical in-app settings surface and optional tray visibility on every desktop
platform; it also includes AppImage-first integration, Linux ARM64 artifacts,
AppImage native-host repair, macOS Dock activation repair, and the package-aware
updater policy. Android
source uses a native Kotlin HTTP server, and the former unpublished
Node/TypeScript CLI and engine have been retired. GitHub release
`android-v0.2.1` contains the signed native-Kotlin APK/AAB with the physically
accepted ChromeOS LAN-address correction. The maintainer reports the exact
Android `v0.2.1` and extension `v0.1.4` candidates submitted to their stores;
public `extension-v0.1.6` is the tested replacement candidate with the
ChromeOS Linux controller and corrected peer-choice copy. Production may still
serve earlier artifacts until review and rollout finish.

## Architecture

Current repository shape:

- `packages/ui` — shared React controls used by the desktop Tauri webview.
- `android` — Compose app with a Kotlin HTTP/storage core and native Android
  lifecycle, permission, background, wake, boot, and battery policy.
- `ios` — independent SwiftUI controls, Swift HTTP/storage code, security-scoped
  Files access, and an intentionally foreground-only lifecycle.
- `desktop` — Tauri app with a Tauri-independent Rust HTTP core and a thin
  React/Tauri command/event control layer.
- `desktop/crostini` — independently released ChromeOS Linux
  launcher/controller that reuses the Rust core.
- `extension` — Published launcher/status surface.

Do not recreate the deleted generic TypeScript native-I/O architecture.
Android and desktop own their Kotlin and Rust implementations respectively;
keep the `desktop/core/src/main.rs` executable repository-only for development
and smoke testing rather than packaging or versioning it as a separate CLI.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kzahel/web-server-chrome](https://github.com/kzahel/web-server-chrome) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
