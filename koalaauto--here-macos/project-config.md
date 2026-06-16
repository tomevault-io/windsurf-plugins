---
trigger: always_on
description: Long-term menu bar macOS app showing your current egress country/region. Default IP-lookup provider: **ipwho.is** (since v0.26.0; previously ip.guide v0.23.x–v0.25.x). The lookup layer is provider-pluggable — see `Here/Networking/IPProvider.swift`.
---

# Here — AI Assistant Notes

Long-term menu bar macOS app showing your current egress country/region. Default IP-lookup provider: **ipwho.is** (since v0.26.0; previously ip.guide v0.23.x–v0.25.x). The lookup layer is provider-pluggable — see `Here/Networking/IPProvider.swift`.

(Originally shipped as "IP Guide" through v0.23.x; renamed to **Here** at v0.24.0. Default provider switched to ipwho.is at v0.26.0 because ip.guide silently misreported VPN egresses — Korean nodes shown as Cyprus, etc.)

## Architecture at a glance

- Entry: `Here/App/HereApp.swift` (SwiftUI `App`, settings-only scene)
- `AppDelegate` builds `AppEnvironment` + `StatusBarController`
- `StatusBarController` owns `NSStatusItem` + `NSPopover` (AppKit-managed)
- Popover content rendered with SwiftUI via `NSHostingController`
- All IP-lookup networking flows through `IPService` (actor) → an `IPProvider` (`IPWhoIsProvider` is the default); emits `IPState` via `AsyncStream`. Each provider owns its own raw-response shape and a `map(_:)` adapter into the shared `IPDataModel` — swapping providers is mechanical, not a UI / cache rewrite.
- `RefreshScheduler` runs the IP refresh loop on a hardcoded 5 s cadence (30 s while the display is asleep). `NetworkMonitor` (`NWPathMonitor`) fires extra immediate refreshes on `becameReachable` / `pathChanged`; lid-wake does the same. The polling loop is the safety net for everything else — at 5 s, you don't need a custom `SCDynamicStore` observer to catch WiFi hops or proxy toggles.
- `UpdateChecker` (actor) + `UpdateInstaller` (actor) + `UpdateCoordinator` (@MainActor) handle the auto-update pipeline.
  - **Checker**: hits GitHub `releases/latest`, returns `UpdateInfo` (version + release URL + DMG asset URL).
  - **Coordinator**: owns the daily wake-up timer, persists `lastUpdateCheckAt` / `skippedUpdateVersion` in `SettingsStore`, presents the "Update available" alert and the install progress NSPanel.
  - **Installer**: URLSession-downloads the DMG (intentionally avoids `com.apple.quarantine` xattr that browser downloads carry — that's how we skip the Gatekeeper "open anyway" prompt on every upgrade), `hdiutil attach`s it, `ditto`-copies `Here.app` to a staging dir, then writes a tiny bash relauncher that polls our PID, swaps `/Applications/Here.app`, and `open`s the new bundle. We `NSApp.terminate(nil)` ourselves; the script outlives us by being detached from our stdio.
- Picker (Never / Once a day / Once a week) + Check now button live in General settings.
- The popover footer's settings gear opens Settings via SwiftUI's `\.openSettings` env action; the right-click menu's Settings… item calls the same captured action through `AppEnvironment.openSettingsAction`. Don't try to open Settings via `NSApp.sendAction(showSettingsWindow:)` from AppKit code — for LSUIElement apps with no visible main menu it silently no-ops.
- Settings use an `@Observable` `SettingsStore` with UserDefaults-backed properties (manual `didSet` persistence)
- Cache at `~/Library/Application Support/Here/last_ip.json` (we ship without App Sandbox — see `Here/Here.entitlements` for why; the in-app installer needs to spawn `hdiutil` and write outside the container).

## Prerequisites

- **Xcode 16+** (only Command Line Tools won't build an `.app` bundle)
- **macOS 15+** runtime
- Swift 6 strict concurrency enabled

Install full Xcode from the Mac App Store or developer.apple.com. `xcode-select -s /Applications/Xcode.app/Contents/Developer` points `xcodebuild` at it.

## Build & run

- Open `Here.xcodeproj` in Xcode, Cmd-R.
- CLI: `xcodebuild -project Here.xcodeproj -scheme Here -configuration Debug build`
- Unit tests: Cmd-U in Xcode, or `xcodebuild -project Here.xcodeproj -scheme Here test`
- Tail logs: `log stream --predicate 'subsystem == "app.here-macos"' --info --debug`

## Conventions

- Swift 6 strict concurrency — actors for shared mutable state, `@MainActor` for UI.
- One concern per file; target under ~200 LOC.
- No third-party UI libraries. Core Services only (`URLSession`, `NWPathMonitor`, `SCDynamicStore`, `CLGeocoder`, `SMAppService`, `NSStatusItem`, `NSPopover`, `MapKit`).
- Prefer `async/await` over Combine.
- Log with `os.Logger` (via `Log` enum), never `print`. IP addresses masked (`.private`) in logs.
- All user-visible strings via `String(localized:)` → `Here/Resources/Localization/en.lproj/Localizable.strings`.
- Tests use Swift Testing (`import Testing`) with `@Test` and `#expect`.

## Known gotchas

- `SMAppService.mainApp.register()` works from any signed bundle path on macOS 13+, including Debug builds running from DerivedData. Earlier-era "must live in /Applications" guidance is no longer accurate; don't re-add that gate. Surface registration errors inline only when `register()` actually throws.
- IP providers don't ship ISO 3166-2 region codes. `RegionMapper` uses `CLGeocoder` with a city-initials fallback. See `Services/RegionMapper.swift` for the ordering.
- Flag emoji for Taiwan (TW) may render as "TW" text on some system configurations — offer text fallback via `CountryStyle.text`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [koalaauto/here-macos](https://github.com/koalaauto/here-macos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
