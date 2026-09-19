---
trigger: always_on
description: Open-source (GPL-3.0) Android app for the AYN Thor that replaces Thor Wayfinder:
---

# Thor Pathfinder — project notes

Open-source (GPL-3.0) Android app for the AYN Thor that replaces Thor Wayfinder:
swaps apps between the two screens and puts shortcuts on the Thor's buttons.
Kotlin + Jetpack Compose, package `com.thorpathfinder.app`, minSdk 33
(the Thor ships Android 13). README.md is the user-facing description.

Machine-specific notes (toolchain paths, the test Thor's serial, git
identity) live in `CLAUDE.local.md`, which is gitignored.

## Status (2026-09-18)

- v0.4.0 (versionCode 9), released 2026-09-18: adds the Screen record action
  (labelled "(Testing)"), the AYN button's own menu when a gesture is left on
  Normal, Check For Updates, and controller scrolling that reaches the true
  top and bottom of a page. 0.3.0 added Close all apps; 0.2.0 was the first
  public release.
  Release APK is about 1.7 MB (R8 on). Bump `versionCode` for every APK
  handed over.
- **Tested on an AYN Thor (firmware 1.0.0.377):** the setup wizard, hold
  Back to swap (two apps; a lone app in both directions), double Back for
  recents, double Select for mouse mode, the focus fix, the YouTube/Discord
  sequence (a lone app's vacated screen goes home, with no flash), Close all
  apps (Recents left empty, apps stopped, screens home, service alive), the
  overlay messages, Screen record from a tile on either Quick Settings page,
  the AYN button with a double-press shortcut (press opens AYN's menu, long
  press its panel), Check For Updates and its release-page link, and
  controller scrolling to both ends of the settings and the shortcut picker.
- **Not yet verified on hardware:** "Open an app", shortcuts on Home in daily
  use, the covered-app fix-up (`moveTaskToFront`), the
  media-first swap order with a real video (logic unit-tested), and the
  "Restricted setting" flow for sideloaded installs (adb installs skip it).
- **Signing.** Releases are signed with the project key (certificate SHA-256
  `2706e85ba69b3f77e37227b4c0a0f99311fc73434d7264216f7bf32cbe439897`,
  CN=Thor Pathfinder). Gradle reads it from `keystore.properties` in the
  project root (gitignored) or from `PATHFINDER_KEYSTORE*` environment
  variables; debug builds use it too, so either installs over the other.
  Without it, both fall back to the local debug key. Never commit, print or
  share the key or its password.
- **Releasing.** Bump `versionCode`/`versionName`, run the tests, build
  `assembleRelease`, copy the APK to `release/Thor-Pathfinder-X.Y.Z.apk`
  (gitignored), check it with `apksigner verify --print-certs`, commit and
  push, then `gh release create vX.Y.Z <apk>` with notes that list the APK's
  SHA-256 and the certificate fingerprint.
- **Clean-room.** Written from scratch. Never copy Thor Wayfinder's code, text
  or branding: it is CC BY-NC-ND, and its README forbids copying, modifying or
  derivatives without written permission. Facts about AYN's firmware are fine.
- **Licenses.** Anything bundled into the APK needs its license in
  `app/src/main/assets/licenses` (shown under About → Open-source licenses)
  and in THIRD_PARTY_NOTICES.md. Shizuku's app terms forbid using the name
  "Shizuku" for an app, its package ID or icon, or declaring its
  `moe.shizuku.manager.permission.*` permissions (using them, as the API
  library does, is how clients work).
- Git: `main`, published at https://github.com/KaitonGxx/thor-pathfinder.

## Layout

```
app/src/main/kotlin/com/thorpathfinder/app/
  Buttons.kt            PhysicalButton (scan codes), Gesture, ButtonAction
  GestureEngine.kt      press / double-press / hold state machine (pure, tested)
  Shortcuts.kt          SharedPreferences store + defaults; implements GestureConfig
  PathfinderService.kt  accessibility service: key events -> engine -> actions
  ScreenSwap.kt         parse `am stack list`, plan, script, covered-app fix-up
  RecentTasks.kt        Close all apps: parse `dumpsys activity recents`, `am stack remove`
  KeyReplay.kt          replay a real key with `sendevent` (the AYN button's own menu)
  ScreenRecord.kt       Screen record: expand QS, `uiautomator dump`, find the tile, tap
  MouseMode.kt          mouse mode toggle, reverse scrolling (AYN config edit)
  UpdateCheck.kt        Check For Updates: GitHub's latest release, version compare
  Shell.kt              Shizuku process runner (newProcess via reflection)
  Overlay.kt            toast-like message as an accessibility overlay window
  Device.kt             Thor + firmware gate (min 1.0.0.377)
  SystemState.kt        what setup checks (device, service, Shizuku, Wayfinder)
  ui/Setup.kt           step-by-step wizard with gated Next
  ui/SettingsScreen.kt  settings; ObservedShortcuts keeps focus on edits
  ui/Components.kt      theme, focusOutline, SwitchRow, ValueRow, ChoiceDialog
  ui/EdgeScroll.kt      ScrollingColumn: controller focus reaches the true ends
  ui/AppPicker.kt       launcher apps for "Open an app"
  ui/Licenses.kt        About → Open-source licenses (texts in assets/licenses)
  ui/Preview.kt         debug builds only: fake states for screenshots
app/src/test/           JVM tests; resources are real captures from the Thor
```

## Facts learned on the Thor (firmware 1.0.0.377)

- **Buttons.** Controller is "Odin Controller" (`/dev/input/event9`):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KaitonGxx/thor-pathfinder](https://github.com/KaitonGxx/thor-pathfinder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-19 -->
