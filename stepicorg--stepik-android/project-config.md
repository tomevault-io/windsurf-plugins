---
trigger: always_on
description: handles the `stepic://.../oauth` flow.
---

# Stepik Android Agent Guide

Last reviewed: 2026-04-28.

## Project Overview

Stepik Android is the open-source native Android client for Stepik, an online
learning platform. The app is distributed on Google Play and the README keeps
store links, screenshots, build/coverage badges, and Apache 2.0 license notes.

- Application id: `org.stepic.droid`.
- Main application class: `app/src/main/java/org/stepic/droid/base/App.kt`.
- Main manifest: `app/src/main/AndroidManifest.xml`.
- The codebase is mostly Kotlin with some Java and has both legacy `org.stepic.droid`
  packages and newer `org.stepik.android` layered packages.
- Build configuration is centralized in `dependencies.gradle`; root `build.gradle`
  wires repositories, Gradle plugins, and shared resolution rules.

## Gradle Modules

The project is declared in `settings.gradle`:

- `:app` - main Android application module. Owns UI, DI graph, networking,
  persistence, resources, assets, manifests, build types, and release packaging.
- `:model` - Android library with shared Stepik model/DTO classes, including
  many `Parcelable` models used across app features.
- `:billing` - Android library around Google Play Billing. Contains billing
  data/domain/remote layers and a Dagger billing component used by `:app`.
- `:androidsvg` - vendored AndroidSVG library module under `com.caverock.androidsvg`.

## Build Stack

Important versions live in `dependencies.gradle`.

- Gradle wrapper: `7.2`.
- Android Gradle Plugin: `7.1.2`.
- Kotlin: `1.7.10`.
- SDKs: `minSdk 21`, `targetSdk 34`, `compileSdk 34`, build tools `34.0.0`.
- Java/Kotlin target: JVM 1.8.
- AndroidX is enabled in `gradle.properties`.
- `:app` uses multidex, ViewBinding, and the deprecated
  `kotlin-android-extensions` plugin. Synthetic view imports and old
  `kotlinx.android.parcel` usage still exist, so Kotlin upgrades need care.
- Repositories include Google, Maven Central, JCenter, JitPack, flatDir AARs,
  and private GitHub Packages for `AndroidKit`, `StoriesKit`, and ktlint rules.
  Private package access may need `GITHUB_USER` and `GITHUB_PERSONAL_ACCESS_TOKEN`.

## Build Types

Declared in `app/build.gradle`:

- `debug` - debug signing, `-debug` suffix, debug endpoint/config support.
- `release` - production signing, minify/shrink enabled, Crashlytics native
  symbol upload configured.
- `releaseOldKeys` - release-like build signed with old keys and
  `BuildConfig.IS_GOOGLE_PLAY=false`.
- `stage` - release-like build signed with old production keys.
- `stageDebuggable` - debuggable variant based on `stage`, with debug endpoint
  support and `-debuggable` suffix.

## Root Structure

Important directories:

```text
.
|-- androidsvg/              # Vendored AndroidSVG library module
|-- app/                     # Main Android application module
|   |-- libs/                # Local AAR/JAR artifacts
|   |-- src/
|   |   |-- main/
|   |   |   |-- AndroidManifest.xml
|   |   |   |-- java/        # App source root
|   |   |   |   |-- org/stepic/droid/   # Legacy app infrastructure and older features
|   |   |   |   `-- org/stepik/android/ # Layered feature code: remote/cache/data/domain/presentation/view
|   |   |   |-- res/         # Layouts, themes, strings, drawables, XML
|   |   |   `-- assets/      # Web, KaTeX, CSS/JS, images, animations, configs
|   |   |-- debug/           # Debug-only code, endpoint switching, assets
|   |   |-- stageDebuggable/ # Stage-debuggable-only code/assets
|   |   |-- release/         # Release build-type implementations
|   |   |-- stage/           # Stage build-type implementations
|   |   |-- releaseOldKeys/  # Old-key release implementations
|   |   |-- test/            # JVM unit tests/resources
|   |   |-- androidTest/     # Instrumented tests
|   |   `-- sharedTest/      # Sources shared by unit and android tests
|   `-- build.gradle         # App build types, plugins, dependencies
|-- billing/                 # Google Play Billing library module
|-- code_quality_tools/      # ktlint, Checkstyle, PMD, Jacoco, FindBugs scripts
|-- docs/                    # Project notes and migration plans
|-- fastlane/                # Google Play alpha deployment lane
|-- gradle/                  # Gradle wrapper files
|-- model/                   # Shared Stepik model/DTO library module
|-- reports/                 # Generated local reports
|-- screenshots/             # README/store screenshots
|-- build.gradle             # Root Gradle build and repositories
|-- dependencies.gradle      # Central versions and dependency coordinates
|-- gradle.properties        # AndroidX/Kotlin/Gradle-wide settings
`-- settings.gradle          # Module list
```

## App Manifest Summary

`app/src/main/AndroidManifest.xml` defines the app shell and external entry points.

- Application class: `.base.App`.
- Launcher entry: `activity-alias org.stepic.droid.view.activities.SplashActivity`
  targeting `org.stepic.droid.ui.activities.SplashActivity`.
- Main navigation: `org.stepic.droid.ui.activities.MainFeedActivity`.
- Auth callbacks: `org.stepik.android.view.auth.ui.activity.SocialAuthActivity`
  handles the `stepic://.../oauth` flow.
- Deep links/App Links cover Stepik hosts for catalog, notifications, stories,
  courses, lessons/steps, users, and review sessions.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [StepicOrg/stepik-android](https://github.com/StepicOrg/stepik-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
