---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

`fanfan` is a macOS menu bar app (Swift 6 / SwiftUI on an AppKit menu-bar lifecycle) that controls Mac fan speeds. It targets **macOS 26+**, Apple Silicon or Intel, and has **no third-party Swift dependencies**.

The repo also contains a small **C daemon** (`fanfan-smcd`) that runs as a root LaunchDaemon. The Swift app is unprivileged; all root-only SMC writes go through the daemon via a Unix socket.

## Privilege architecture — read this before touching fan-write code

```
fanfan.app (user)  ──Unix socket──▶  fanfan-smcd (root)  ──IOKit──▶  AppleSMC
```

- **Temperature / sensor reads** happen directly from the app via IOKit. No daemon needed, no elevated privileges.
- **Fan writes** (`SET`, `AUTO`) must go through the daemon. The socket protocol is intentionally tiny — only three commands ever cross it:
  - `PING` → `OK pong`
  - `SET <fan> <rpm>` (fan: 0–7; rpm: 500–8000, enforced in daemon)
  - `AUTO <fan>` (hand control back to firmware)
- Socket path: `/var/run/fanfan-smcd.sock` (mode `0660`, owner `root:admin`).
- Daemon binary lives at `/usr/local/libexec/fanfan-smcd`; LaunchDaemon plist at `/Library/LaunchDaemons/com.hoobnn.fanfan.smcd.plist`.
- Installation is performed by `PermissionsManager.installHelper` via a single `osascript` "with administrator privileges" call — one password prompt, ever. Do not introduce additional sudo prompts or new commands across the socket without strong justification: the small, audited surface is a deliberate security property.

Swift side of the socket: `fanfan/Core/SMCDaemonClient.swift`. Daemon source: `tools/fanfan-smcd/fanfan-smcd.c` (+ `smc.h`). If you change one, the other almost always needs a matching change.

## Code layout (only the non-obvious parts)

- `fanfan/App/fanfanApp.swift` — `@main` SwiftUI `App` + `AppDelegate`. The app uses `NSApp.setActivationPolicy(.accessory)` (no Dock icon) and a `Settings` scene placeholder; the real UI surfaces are the status-bar popover (`StatusBarManager`) and a separate `Window("settings")` scene.
- `fanfan/Core/`
  - `SystemMonitor.swift` — IOKit-backed sensor/SMC reader (large; the catalogue of SMC keys lives here).
  - `FanController.swift` — automatic-mode controller: rolling temperature history, ±200 RPM hysteresis, smooth ramping, and the Silent / Balanced / Performance / Custom presets.
  - `StatusBarManager.swift` — menu-bar item, popover lifecycle, and the four status-bar display modes (`temperature`, `power`, `fanPercent`, `iconOnly`) driven by the `StatusBarDisplayModeChanged` notification.
  - `SMCDaemonClient.swift`, `PermissionsManager.swift` — see "Privilege architecture" above.
  - `BatteryMonitor.swift`, `LaunchAtLoginManager.swift` (uses `ServiceManager`, no plist hacks), `UserDefaultsManager.swift`, `FanRPMBounds.swift`.
- `fanfan/ViewModels/FanControlViewModel.swift` — the single Combine `ObservableObject` wiring `SystemMonitor` + `FanController` + user defaults to the views. UI mutations flow through this; treat it as the source of truth for fan state in the UI layer.
- `fanfan/UI/Theme.swift` — design rule the codebase follows literally: **temperature is the only color**. The whole UI is monochrome and a near-invisible warm tint bleeds in from the top of the popover as temps rise. When adding UI, pull colors / accents from `Theme`, do not hardcode hues.
- `fanfan/UI/Views/` — `PopoverView` (menu-bar popover) and `SettingsView`. Reusable bits under `Components/` (`HeroCard`, `ControlsCard`, `HeatBar`, `TempCurveView`, `FanBladeView`, …).
- `fanfan/Resources/`
  - `Assets.xcassets`, localized strings under `en.lproj/` and `zh-Hans.lproj/` (use `NSLocalizedString` — both locales must stay in sync).
  - `fanfan-smcd` (the compiled C daemon, bundled as a resource) and `com.hoobnn.fanfan.smcd.plist`.
  - `fanfan.entitlements` (hardened runtime entitlements for Developer ID signing).
- `tools/fanfan-smcd/` — C daemon source + `Makefile` + `smc.h`.
- `fanfanTests/` — `FanControlTests.swift`, `SystemMonitorTests.swift`, `fanfanTests.swift` (XCTest target).
- `scripts/build-release.sh` — full Developer ID release pipeline (build → sign → notarize → staple → zip + DMG + sha256). `scripts/install.sh` is the public one-liner installer.
- `.github/workflows/release.yml` — CI tag-triggered release build (`v*` tags or manual dispatch).

## Build, run, test

This is an Xcode project — there is no `swift build` / SwiftPM. Use the Xcode CLI:

```bash
# Build (Debug) headlessly
xcodebuild -project fanfan.xcodeproj -scheme fanfan -configuration Debug build

# Run the test bundle
xcodebuild -project fanfan.xcodeproj -scheme fanfan \
  -destination 'platform=macOS' test

# Run a single test method
xcodebuild -project fanfan.xcodeproj -scheme fanfan \
  -destination 'platform=macOS' test \
  -only-testing:fanfanTests/FanControlTests/<methodName>
```

For interactive development, open `fanfan.xcodeproj` in Xcode and use the `fanfan` scheme.

### Building the daemon standalone

```bash
make -C tools/fanfan-smcd            # produces tools/fanfan-smcd/fanfan-smcd
make -C tools/fanfan-smcd clean
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hoobnn/fanfan](https://github.com/hoobnn/fanfan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
