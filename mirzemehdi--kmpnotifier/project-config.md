---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

KMPNotifier is a Kotlin Multiplatform notification library (`io.github.mirzemehdi`) for local notifications (Android, iOS, desktop/JVM, JS, wasmJS) and Firebase push notifications (Android/iOS only). Gradle modules:

- `:kmpnotifier-core` — shared core: `KMPNotifier` facade, configuration, permissions, logger, manual dependency wiring, internal event hub. All targets.
- `:kmpnotifier-local` — local notifications (`LocalNotifications` extension, `Notifier` + platform impls). All targets. Depends on core.
- `:kmpnotifier-push-firebase` — Firebase push (`FirebasePush` extension, `PushListener`). All targets — Firebase delivery on android/iOS, shared no-op mock elsewhere (1.x parity). Depends on local. Declares the Firebase iOS dependency via SwiftPM (`swiftPMDependencies`).
- `:kmpnotifier` (directory `deprecated/kmpnotifier`, mapped via settings `projectDir`) — deprecated compatibility umbrella: old `NotifierManager` API forwarding to the new API; `api()`-depends on all modules. Publishing is gated behind `-PpublishDeprecatedUmbrella=true` (2.0.0 ships it once via the CI release step; remove that flag from the workflow afterwards — later versions must not publish it). Removal planned for 3.0.0.
- Demo app (JetBrains default structure, AGP 9 Path C): `:shared` — Compose Multiplatform shared UI + platform actuals + the iOS framework (baseName `shared`; no iosX64 — CMP 1.11+ has no Intel-simulator artifacts); `:androidApp`, `:desktopApp`, `:webApp` — per-platform entry points; `iosApp/` — Xcode project (its build phase runs `:shared:embedAndSignAppleFrameworkForXcode`)

All four library modules apply the `kmpnotifier.library` convention plugin from the `build-logic/` included build (targets incl. the AGP 9 `com.android.kotlin.multiplatform.library` android target — single variant, so no `-android-debug` artifacts; shared test deps; Maven Central publishing). Android unit tests live in `src/androidHostTest/`. Module build files declare only: dependencies, `android.namespace`, POM name/description, and module-specific blocks (push: `swiftPMDependencies`).

## Commands

JDK 17+ required (e.g. `JAVA_HOME=$(/usr/libexec/java_home -v 17)`).

```sh
./gradlew apiCheck                                 # binary compatibility check (runs first in CI)
./gradlew apiDump                                  # regenerate <module>/api/* after intentional public API changes (run on macOS)
./gradlew testAndroidHostTest                      # Android host (unit) tests incl. Robolectric, all modules
./gradlew jvmTest jsNodeTest                       # desktop + js tests
./gradlew iosX64Test iosSimulatorArm64Test         # iOS tests (macOS only)
./gradlew :kmpnotifier-core:jvmTest --tests "com.mmk.kmpnotifier.SomeTest"  # single test class
./gradlew :dokkaGenerate                           # docs site (Dokka V2 aggregate, output build/dokka/html; published to GitHub Pages on release)
./gradlew publishToMavenLocal                      # local publish (signing is skipped for this task)
./gradlew kotlinUpgradeYarnLock                    # after js dependency changes (yarn.lock check fails otherwise)
```

Sample app:

```sh
./gradlew :androidApp:installDebug                 # Android
./gradlew :desktopApp:run                          # Desktop
./gradlew :webApp:wasmJsBrowserDevelopmentRun      # Web (wasm)
```

`firebase_push_notification_sender.py` sends test FCM pushes (requires a Firebase service-account JSON and project ID filled in at the top of the script).

## Releasing

Version is `kmpNotifierVersion` in `gradle.properties` (all four artifacts share it). Pushing a `v*` tag triggers CI release: Dokka docs to GitHub Pages + `publishAndReleaseToMavenCentral`. CI runs `apiCheck` before any build, so a public API change without a committed `apiDump` fails the pipeline. artifactId == Gradle project name (set by the convention plugin) — renaming a module directory changes the published coordinates.

## Architecture

**Facade + pluggable extensions.** `KMPNotifier.initialize(configuration, vararg extensions)` (core) wires dependencies manually (no DI framework) and installs `KMPNotifierExtension`s. `LocalNotifications` (local module) registers the platform `Notifier`; `FirebasePush` (push module) registers the Firebase `PushNotifier` and declares `dependsOn = [LocalNotifications]`, so users pass only `FirebasePush`. Accessors: `KMPNotifier.localNotifier` / `LocalNotifications.notifier`, `KMPNotifier.firebasePushNotifier` / `FirebasePush.notifier` (firebase delivery android/ios; no-op mock on jvm/js/wasm).

**Cross-module wiring via `@InternalKMPNotifierApi`** (`com.mmk.kmpnotifier.internal`, opt-in ERROR level):
- `NotifierInternals` — registries (local notifier as `Any` since `Notifier` lives downstream; push notifier), configuration/permission access, `resetForTests()`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mirzemehdi/KMPNotifier](https://github.com/mirzemehdi/KMPNotifier) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
