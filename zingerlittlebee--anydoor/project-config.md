---
trigger: always_on
description: <!-- This file mirrors CLAUDE.md; keep the two in sync when either changes. -->
---

<!-- This file mirrors CLAUDE.md; keep the two in sync when either changes. -->

# AnyDoor

A macOS menu-bar toolbox. At its core it toggles (show/hide) a target application via a global hotkey, and builds a set of system-level quick actions on top of that: clipboard history, hosts management, external display brightness, Hyper Key, window layout, command palette, and more.

## Tech Stack

- Swift 6.2, strict concurrency mode (`.swiftLanguageMode(.v6)`)
- macOS 14+
- Liquid Glass on macOS 26+; earlier supported systems fall back to a plain material background.
- SwiftUI `Settings` scene + AppKit `MenuBarController` (the menu-bar item is managed directly by `NSStatusItem`, **not** `MenuBarExtra` — see below)
- SwiftData persistence
- CGEvent tap for global hotkey monitoring
- Privileged XPC helper (`AnyDoorHostsHelper`) writes `/etc/hosts` (with an AppleScript fallback when the helper isn't enabled — see Architecture Notes)
- Sparkle for auto-updates; MonitorControl's MIT `IntelDDC` (vendored under `Services/Brightness/Vendor/`) for Intel external-display brightness; AskForPermission for permission onboarding. Bundled third-party license texts live in `THIRD-PARTY-LICENSES.md`.
- SPM build, with an in-repo build-tool plugin that compiles the `.xcstrings` string catalog

## Build and Run

```bash
# Build
swift build

# Run (dev mode; the process has no Bundle ID identity)
swift run AnyDoor

# Release build
swift build -c release

# Install as /Applications/AnyDoor.app (writes Info.plist, Bundle ID = dev.bybee.AnyDoor)
make install

# Uninstall
make uninstall

# Hot-reload development (requires watchexec)
make
```

Running requires the macOS Accessibility permission (System Settings → Privacy & Security → Accessibility).

The `.app` from `swift run` and the one from `make install` are **two distinct process identities** and must each be granted Accessibility separately. Use `make install` for production; use `swift run` for daily development. The SwiftData store path is pinned so both paths share the same data (see below).

Release tooling lives in the `Makefile`: `make sparkle-tools` (downloads the Sparkle CLI, pinned to 2.9.2), `make release <version>` / `make release-dryrun <version>` (drive `scripts/release.sh`, sourcing `.env`), and `make notary-profile` / `make notary-check` (notarization via `xcrun notarytool`, requires `APPLE_ID` / `APPLE_TEAM_ID` / `NOTARY_PROFILE` in `.env`). Release notes go under `## [Unreleased]` in `CHANGELOG.md` (do not hand-author a `## [x.y.z]` heading); `scripts/release.sh` rewrites that heading to `## [<version>] - <today>` on release. Its preflight (step 1) asserts: `[Unreleased]` is non-empty, `Info.plist` `SUPublicEDKey` is not a placeholder (run `scripts/sparkle-bin/generate_keys` first), a codesigning cert matching `SIGNING_IDENTITY` is in the login keychain, `sign_update` / `generate_appcast` are installed (`make sparkle-tools`), the `notarytool` keychain profile is configured, and `gh` is logged in.

## Project Structure

The codebase is large; the layout below is organized by subsystem (not a file-by-file listing). SPM targets: `AnyDoor` (main app), `HostsHelperShared` (shared XPC-contract library), `XPCAuditToken` (ObjC shim exposing the XPC peer audit token), `AnyDoorHostsHelper` (privileged helper executable), `AnyDoorTests`, and `XCStringsCompilerPlugin` (a `.buildTool()` plugin at `Plugins/XCStringsCompiler`, attached to `AnyDoor`, that compiles `.xcstrings` string catalogs via `xcrun xcstringstool` at build time — this is why localization works under plain `swift build` without Xcode).

```
Sources/AnyDoor/
├── AnyDoor.swift               # @main, Settings scene only (menu bar is managed by MenuBarController)
├── AppDelegate.swift           # ModelContainer, providers registry, service bootstrap, state-restoration opt-out
├── Models/                     # SwiftData @Model (exactly these 4 = the ModelContainer schema):
│                               #   KeyBinding / BuiltinPreference / ClipboardHistoryItem / HostProfile.
│                               #   Value types: BuiltinItem / PanelEntry (+ HotkeyDescriptor) /
│                               #   HotkeyAction (+ HotkeySnapshot) / HyperKey.swift (HyperKeyTrigger /
│                               #   HyperKeyQuickPress / HyperKeyVirtualKey) / PortRecord / MenuBarIcon /
│                               #   BackupSnapshot (Codable backup DTO, NOT a SwiftData @Model)
├── Services/
│   ├── Core         HotkeyService / PanelStore / AppSwitcher / MenuBarController /
│   │                SettingsOpener / RegularWindowCoordinator / LaunchAtLogin / LocalizationManager
│   ├── Seeding      BuiltinPreferenceSeeder / KeyBindingOrderBackfill (idempotent launch-time migrations)
│   ├── Runners      AppleScriptRunner / ShellRunner / CommandRunner / AutomationPermission
│   ├── Providers/   23 ToggleProvider/ActionProvider types (BuiltinProvider.swift holds the protocols);
│   │                most are their own actor (see Architecture Notes)
│   ├── Clipboard    ClipboardWatcher / ClipboardHistoryStore / ClipboardCapture / ClipboardPasteService /
│   │                ClipboardSearch / ClipboardPreferences / ColorSampler / TextRecognizer /
│   │                BarcodeRecognizer / RegionCapture

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZingerLittleBee/AnyDoor](https://github.com/ZingerLittleBee/AnyDoor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
