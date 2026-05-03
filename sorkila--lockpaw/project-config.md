---
trigger: always_on
description: macOS menu bar screen guard. Lock/unlock with a hotkey. Dog mascot.
---

# Lockpaw

macOS menu bar screen guard. Lock/unlock with a hotkey. Dog mascot.

## Quick reference

- **App name:** Lockpaw
- **Bundle ID:** `com.eriknielsen.lockpaw`
- **URL scheme:** `lockpaw://`
- **Website:** getlockpaw.com (hosted on Inleed, deployed via FTP from `sorkila/lockpaw-web`)
- **Repo:** git@github.com:sorkila/lockpaw.git
- **Requires:** macOS 14+, Xcode 16+, XcodeGen
- **Dependencies:** Sparkle (SPM, auto-updates with EdDSA signing)
- **Current version:** 1.0.7

## Build

```bash
xcodegen generate
xcodebuild -project Lockpaw.xcodeproj -scheme Lockpaw -configuration Debug build
```

After each rebuild, reset TCC (binary signature changes invalidate accessibility permission):
```bash
tccutil reset Accessibility com.eriknielsen.lockpaw
```

## Test

```bash
xcodebuild -project Lockpaw.xcodeproj -scheme Lockpaw -configuration Debug test
```

34 unit tests covering LockState transitions, Constants formatting, and HotkeyConfig conflict detection.

## Release

```bash
./scripts/build-release.sh
```

Builds unsigned → copies to `/tmp` for signing → signs with Developer ID → creates DMG → notarizes → staples → sets custom DMG file icon. Output: `build/Lockpaw.dmg`.

**Requires:** `lockpaw-notarize` keychain profile (already stored), Sparkle EdDSA signing key in Keychain.

**Signing:** The build script copies the app to `/tmp` via `ditto --norsrc` before signing. This is required because the repo lives in iCloud-synced `~/Documents` which adds irremovable `com.apple.FinderInfo` and `com.apple.fileprovider.fpfs#P` xattrs that cause codesign to fail with "resource fork, Finder information, or similar detritus not allowed". Signing is done inside-out with `--timestamp`: XPC service binaries → XPC bundles → Autoupdate → Updater.app binary → Updater.app → Sparkle.framework → main app.

**DMG pipeline:** Builds a R/W DMG via `hdiutil`, copies app + Finder alias (not symlink) to `/Applications`, applies AppleScript window styling (background, icon positions, hide dotfiles), copies volume icon AFTER AppleScript (the `update` command deletes `.VolumeIcon.icns`), then converts once to compressed UDZO. No intermediate conversions.

**After building a release:**
1. Tag: `git tag -a vX.Y.Z -m "..." && git push origin vX.Y.Z`
2. Create GitHub Release with DMG: `gh release create vX.Y.Z build/Lockpaw.dmg#Lockpaw.dmg --repo sorkila/lockpaw`
3. Update appcast: `generate_appcast build/appcast/` → fix download URL to GitHub Releases → push `appcast.xml` to `sorkila/lockpaw-web`
4. Update Homebrew cask SHA256 in both `sorkila/homebrew-lockpaw` and `homebrew/Casks/lockpaw.rb`

**DMG assets** in `scripts/`:
- `dmg-background.png` / `dmg-background@2x.png` — light background with teal arrow (660x480 / 1320x960)
- `dmg-volume-icon.icns` — dog mascot icon shown on mounted volume and DMG file in Finder

**DMG design notes:**
- Uses Finder alias (not symlink) for Applications — symlinks show broken icon on Sonoma+
- AppleScript's `update without registering applications` deletes `.VolumeIcon.icns` — must copy icon AFTER AppleScript
- Dotfiles (`.background`, `.fseventsd`) pushed off-screen via AppleScript positioning
- Light background for readable dark text labels in Finder light mode

## Project structure

```
Lockpaw/
├── LockpawApp.swift                Entry point, MenuBarExtra, AppDelegate, onboarding
├── Controllers/
│   ├── LockController.swift        State machine, lock/unlock orchestration, toggle observer
│   ├── Authenticator.swift         LAContext (Touch ID / password fallback)
│   ├── InputBlocker.swift          CGEventTap — blocks keyboard/scroll while locked
│   ├── HotkeyManager.swift         CGEventTap on dedicated background thread — global hotkey
│   ├── OverlayWindowManager.swift  NSWindow per screen at CGShieldingWindowLevel
│   └── SleepPreventer.swift        IOKit sleep assertion
├── Models/
│   ├── LockState.swift             .unlocked → .locking → .locked → .unlocking
│   └── HotkeyConfig.swift          Centralized hotkey UserDefaults + system conflict detection
├── Views/
│   ├── LockScreenView.swift        Lock screen — dog, message, time, fallback auth
│   ├── AmbientScreenView.swift     Secondary display — morphing gradient blobs
│   ├── MenuBarView.swift           Menu bar dropdown
│   ├── SettingsView.swift          Native Form, hotkey recorder, appearance, UpdateCheckViewModel
│   └── OnboardingView.swift        4 steps: welcome, hotkey, accessibility, menu bar
├── Utilities/
│   ├── Constants.swift             App constants, Timing enum, animation presets, formatting
│   ├── Notifications.swift         All Notification.Name in one place
│   └── AccessibilityChecker.swift  AXIsProcessTrusted + System Settings opener
├── Resources/
│   └── Assets.xcassets             App icon, mascot, menu bar icon (template), colors
└── LockpawTests/
    ├── LockStateTests.swift        State transition validation (16 tests)
    ├── ConstantsTests.swift         Time formatting (11 tests)
    └── HotkeyConfigTests.swift      System shortcut conflict detection (7 tests)
```

## Repo-level directories

- **`assets/`** — `demo.gif` hero GIF for README (lock/unlock flow, 800px wide)
- **`scripts/`** — `build-release.sh`, DMG background PNGs, volume icon

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sorkila/lockpaw](https://github.com/sorkila/lockpaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
