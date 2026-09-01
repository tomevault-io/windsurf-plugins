---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

YATA ("Yet Another Task App") — a Material 3 Expressive task manager for Android, built with Jetpack Compose, Room, and Hilt. Gradle root project name is `TodoExpressive` (legacy); package/app id is `com.mj.yata`. Two modules:
- `:app` — the phone app (`com.mj.yata`, minSdk 26, compileSdk/targetSdk 35).
- `:baselineprofile` — a `com.android.test` module that records the ART baseline profile packaged with the app.

## Commands

All commands run from the repo root using the Gradle wrapper (`./gradlew` on Bash, `gradlew.bat` on native Windows shells).

```bash
# Compile Kotlin only (fast correctness check, no packaging) — use this while iterating
./gradlew :app:compileDebugKotlin -q

# Full debug build
./gradlew :app:assembleDebug -q

# Install to a connected/emulated device
./gradlew :app:installDebug -q

# Unit tests (JVM, no device) — e.g. RecurrenceEvaluatorTest, NaturalLanguageParserTest
./gradlew :app:testDebugUnitTest
./gradlew :app:testDebugUnitTest --tests "com.mj.yata.RecurrenceEvaluatorTest"
./gradlew :app:testDebugUnitTest --tests "com.mj.yata.RecurrenceEvaluatorTest.testWeeklyRecurrence"

# Instrumented tests. DESTRUCTIVE — see the warning below; -PdisposableDevice is required, and the
# build refuses to run without it. Note `--tests` does NOT work here; that is Gradle's unit-test
# filter, and connectedAndroidTest takes a runner argument instead.
./gradlew :app:connectedDebugAndroidTest -PdisposableDevice   # whole suite
./gradlew :app:connectedDebugAndroidTest -PdisposableDevice -Pandroid.testInstrumentationRunnerArguments.class=com.mj.yata.data.local.db.AppDatabaseMigrationTest

# Compose UI smoke suite (launch, add, complete, tab switch, delete-undo)
./gradlew :app:connectedDebugAndroidTest -PdisposableDevice -Pandroid.testInstrumentationRunnerArguments.class=com.mj.yata.MainScreenSmokeTest

# Regenerate the baseline profile (needs a rooted/userdebug device or emulator, API 28+)
./gradlew :baselineprofile:generateBaselineProfile
```

After changing anything under `app/src/main/java`, the fast loop is `compileDebugKotlin` to catch errors, then `installDebug` before manually verifying in the UI. `MainScreenSmokeTest` covers only the core add/complete/delete paths, so anything beyond those still needs a manual pass on-device.

**Never run instrumented tests against the user's phone.** `connectedAndroidTest` is not read-only: it reinstalls the app, and the tests add, complete and delete real rows in whatever database is on the device. It also used to uninstall the app on completion, wiping the Room database and DataStore — which is exactly how a real day's data was destroyed on this project. Two guards now exist: `android.injected.androidTest.leaveApksInstalledAfterRun=true` in `gradle.properties` stops the uninstall, and a `doFirst` check in `app/build.gradle.kts` refuses the task without `-PdisposableDevice`. **Do not pass that flag on the user's device**, and don't work around either guard — they exist because the failure is silent until the data is gone. Emulator or a spare device only, and ask first.

More generally: confirm before any command that can remove the app or its data — `adb uninstall`, `pm clear`, `installDebug` over a different signing key, or anything that resets storage. The user's phone is their daily driver, not a test rig.

**Never drive the device to take screenshots.** Do not use `adb shell screencap`, `uiautomator dump`, `adb shell input tap/swipe/keyevent`, or any other means of navigating the running app to look at it. Build, install, and describe what changed — visual verification is the user's, and they will send a screenshot when they want one. This applies even when a UI change would obviously benefit from being seen.

## Changelog

`CHANGELOG.md` is maintained by hand, newest first, in Keep a Changelog format. **Any change a user
would notice goes under `[Unreleased]` in the same commit that makes it** — not batched up at
release time. Internal refactors, build plumbing and test-only work stay in the commit message
unless they change behaviour.

Releasing: rename `[Unreleased]` to the new version with a date and `versionCode`, open a fresh
empty `[Unreleased]` above it, add the compare/tag links at the bottom, then use that section
verbatim as the GitHub release notes (`gh release create <tag> <apk> --notes-file …`). The release
notes and the changelog should never be written twice.

## Architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rjwarrier/yata](https://github.com/rjwarrier/yata) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
