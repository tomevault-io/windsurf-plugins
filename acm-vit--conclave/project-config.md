---
trigger: always_on
description: This directory is the active native app. Do not make new changes in
---

# Conclave Skip Native Agent Notes

This directory is the active native app. Do not make new changes in
`apps/mobile`; it is deprecated for this workstream.

`apps/conclave-skip` is a Skip app: Swift is the source of truth, iOS compiles
the Swift directly, and Android is generated/transpiled to Kotlin/Jetpack
Compose with a few hand-written Kotlin bridges in `Sources/Conclave/Skip`.
Every change must be judged on both platforms.

## Current Priorities

The native app was improved specifically around meeting entry, Lottie,
bottom sheets, games, and responsiveness. Continue from those assumptions:

- Native should feel instant. Avoid broad recomposition and heavy view trees.
- Android sheets must use the native `FlexibleBottomSheet` bridge, not a custom
  fake sheet.
- Android Lottie must use LottieFiles `dotlottie-android`, not Airbnb
  `lottie-compose` for `.lottie` playback.
- iOS must stay green with `swift build` and `swift test`.
- No gradients anywhere. Use dark flat surfaces, 1 px borders, and the coral
  accent only.
- Do not re-enable R8/minification. Release builds are intentionally
  un-minified because R8 has broken SkipUI runtime behavior.

## Build And Verification Commands

Run these from `/Users/ishaan/Projects/conclave` unless noted.

```bash
cd apps/conclave-skip
swift build
swift test
```

Android uses the system Gradle binary, not a wrapper:

```bash
cd apps/conclave-skip/Android
/opt/homebrew/bin/gradle :Conclave:compileDebugKotlin --console=plain
ALLOW_DEBUG_RELEASE_SIGNING=true /opt/homebrew/bin/gradle :app:assembleRelease --console=plain
```

The local release APK is written to:

```text
apps/conclave-skip/.build/Android/app/outputs/apk/release/app-release.apk
```

For iOS simulator builds, the Skip project may need plugin validation skipped
and Skip actions disabled during Xcode build:

```bash
xcodebuild \
  -project apps/conclave-skip/Darwin/Conclave.xcodeproj \
  -scheme "Conclave App" \
  -configuration Debug \
  -destination 'id=<SIMULATOR_UDID>' \
  -derivedDataPath /tmp/conclave-ios-dd \
  -skipPackagePluginValidation \
  -skipMacroValidation \
  build SKIP_ZERO=1 SKIP_ACTION=none
```

Install and open with `agent-device` when a simulator is available:

```bash
agent-device install com.acmvit.conclave /tmp/conclave-ios-dd/Build/Products/Debug-iphonesimulator/Conclave.app --platform ios --udid <SIMULATOR_UDID>
agent-device open com.acmvit.conclave --platform ios --udid <SIMULATOR_UDID>
```

Debug **simulator** builds default the SFU join endpoint to
`http://127.0.0.1:3000` (local dev stack); a guest join fails with the branded
"Could not connect to the server" screen unless a local stack is running. To
smoke against production instead, launch with the env override:

```bash
xcrun simctl terminate <SIMULATOR_UDID> com.acmvit.conclave
SIMCTL_CHILD_SFU_JOIN_URL="https://conclave.acmvit.in/api/sfu/join" \
  xcrun simctl launch <SIMULATOR_UDID> com.acmvit.conclave
```

(Resolution order lives in `SfuJoinService.joinURL()`: env override → bundled
config → 127.0.0.1 on sim / production on device. Debug on a real device and
all Release builds already point at production.)

`agent-device snapshot` + `click @ref` (accessibility refs) is the reliable
way to drive the app on simulator - prefer it over coordinate taps. Sim smoke
of the core loop (guest create → entry overlay with animating Lottie →
settled meeting → chat/sheet → hang up) passed against production on
2026-07-07.

One more transpile gotcha proven here: `.map(String.init)` (any
`map`/`compactMap` with an initializer or function reference) transpiles to
Kotlin `map(String)` - passing the companion object - and fails
`compileReleaseKotlin`. Always use an explicit closure: `.map { String($0) }`.
This shipped in the chess commits and broke the Android build while iOS stayed
green; Android compile is part of "done" for every commit.

Real Android proof still matters for completion:

```bash
adb devices -l
adb install -r apps/conclave-skip/.build/Android/app/outputs/apk/release/app-release.apk
adb shell uiautomator dump /sdcard/ui.xml
adb pull /sdcard/ui.xml /tmp/conclave-ui.xml
```

Do not use `uiautomator dump /dev/tty`; it can collide with stale
UiAutomation sessions. Dump to a file and pull it.

Do not run two Gradle invocations on this project concurrently (e.g. a
background `:app:assembleRelease` while a foreground `:Conclave:compileDebugKotlin`
or an Xcode build runs). The skip prebuild and shared `.build` outputs
collide and the build fails spuriously ("N actionable tasks: 1 executed" with
no error lines). Re-run sequentially before diagnosing anything.

Prod rate-limits rapid room creation per client: after ~a dozen creates in
one session the app path returns "Join request failed" for a while. Space out
sim smoke runs or reuse one room instead of creating fresh rooms per test.

## Performance Learnings

The app felt sluggish mostly because of over-broad recomposition, not because
of a single expensive CPU task. SkipUI maps SwiftUI view bodies into Compose
functions. If a large body reads `MeetingViewModel.shared` or many fields from
`MeetingState`, unrelated state changes can rebuild large subtrees.

The important fixes and patterns:

- Split large views into narrow leaf views.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ACM-VIT/conclave](https://github.com/ACM-VIT/conclave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
