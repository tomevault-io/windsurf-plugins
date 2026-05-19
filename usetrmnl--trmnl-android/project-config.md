---
trigger: always_on
description: This document provides essential information for GitHub Copilot agents working on the TRMNL Android codebase.
---

# TRMNL Android - Copilot Agent Instructions

This document provides essential information for GitHub Copilot agents working on the TRMNL Android codebase.

## Repository Overview

**TRMNL Android** is a native Android application that displays TRMNL e-ink device content on Android phones, tablets, and e-ink displays. The app connects to TRMNL or BYOS (Bring Your Own Server) APIs, fetches display images periodically, and renders them on the device screen.

**Key Statistics:**
- Language: Kotlin (100%)
- Build System: Gradle 8.13
- Min SDK: 28 (Android 9.0 Pie)
- Target SDK: 36 (Android 16.0)
- Architecture: Modern Android with Jetpack Compose, Circuit UDF, Metro DI

## Critical Build & Test Commands

**ALWAYS run these commands in this specific order to validate changes:**

### 1. Format Code (Required Before Commit)
```bash
./gradlew formatKotlin
```
- Formats all Kotlin code using ktlint
- Must run BEFORE committing code changes
- Takes ~5 seconds
- Zero failures expected

### 2. Lint & Test (Required Before Commit)
```bash
./gradlew lintKotlin testDebugUnitTest --parallel --daemon
```
- Runs Kotlin linting and unit tests
- Takes ~2-3 minutes on first run, ~1 minute with cache
- This is the EXACT command used in CI (`.github/workflows/android.yml`)
- All tests must pass before submitting changes

### 3. Build Debug APK
```bash
./gradlew assembleDebug --parallel --daemon
```
- Builds the debug APK
- Takes ~1-2 minutes with cache
- Output: `app/build/outputs/apk/debug/app-debug.apk`
- Uses debug keystore from `keystore/debug.keystore`

### 4. Android Lint (Post-Merge Validation)
```bash
./gradlew lintDebug
```
- Runs Android lint checks
- Takes ~50 seconds
- Generates HTML report: `app/build/reports/lint-results-debug.html`
- Used in post-merge CI workflow

## Environment Requirements

- **JDK Version:** 21 (OpenJDK 21 - Temurin distribution recommended)
- **Gradle:** 8.13 (via wrapper, do NOT install manually)
- **Android SDK:** Compile SDK 36
- **Build Tools:** Managed by Gradle plugin (AGP 8.9.2)

**IMPORTANT:** Always use `./gradlew` (Gradle wrapper) - NEVER use a system-installed gradle.

## Project Architecture

### Main Source Structure
```
app/src/main/java/ink/trmnl/android/
├── MainActivity.kt               # App entry point (138 lines)
├── TrmnlDisplayMirrorApp.kt     # Application class (51 lines)
├── data/                         # Repositories, DataStore implementations
├── di/                          # Metro dependency injection modules
├── model/                       # Data models (TrmnlDeviceConfig, etc.)
├── network/                     # Retrofit API service, response models
├── ui/                          # Jetpack Compose screens (Circuit UDF)
├── util/                        # Utilities (InputValidator, etc.)
└── work/                        # WorkManager background refresh logic
```

### Key Configuration Files
- `app/build.gradle.kts` - Main build configuration, versioning (versionCode, versionName)
- `build.gradle.kts` - Root project plugins
- `settings.gradle.kts` - Project settings
- `gradle.properties` - Gradle JVM settings (2GB heap)
- `gradle/libs.versions.toml` - Centralized dependency versions
- `app/proguard-rules.pro` - ProGuard rules (mostly default)
- `app/src/main/AndroidManifest.xml` - App manifest, permissions

### Key Architectural Patterns
- **UI Framework:** Jetpack Compose with Circuit (Slack's UDF architecture)
- **DI:** Metro (dev.zacsweers.metro) for compile-time code generation
- **Background Work:** WorkManager (15 min minimum interval)
- **Networking:** Retrofit + OkHttp + Moshi for JSON parsing
- **Data Storage:** DataStore (preferences) for settings/tokens
- **Image Loading:** Coil 3.x with OkHttp integration
- **State Management:** Circuit's presenter pattern with `@CircuitInject`

### Main Features & Screens
- `TrmnlMirrorDisplayScreen` - Main display showing TRMNL image
- `AppSettingsScreen` - Configuration (API token, server URL)
- `DisplayRefreshLogScreen` - Refresh history/logs
- `TrmnlImageRefreshWorker` - Background image refresh job
- `TrmnlWorkScheduler` - Manages WorkManager scheduling

## CI/CD Workflows

All workflows are in `.github/workflows/`:

1. **android.yml** (PR & Main Branch)
   - Triggers: PRs to main, pushes to main
   - Commands: `./gradlew lintKotlin testDebugUnitTest --parallel --daemon` then `./gradlew assembleDebug --parallel --daemon`
   - Takes ~3 minutes total

2. **android-lint.yml** (Post-Merge)
   - Triggers: Pushes to main
   - Commands: `./gradlew lintDebug` then `./gradlew assembleDebug`
   - Additional validation after merge

3. **android-release.yml** (Release Builds)
   - Triggers: Pushes to main, manual, GitHub releases
   - Builds signed release APK and AAB
   - Requires: `KEYSTORE_BASE64`, `KEYSTORE_PASSWORD`, `KEY_ALIAS` secrets
   - Attaches APK to GitHub releases automatically

4. **version-management.yml** (Version Bumping)
   - Manual workflow for updating app version
   - Updates: `app/build.gradle.kts`, changelogs
   - Run `./scripts/show_version_info.sh` locally to get current version

## Common Issues & Workarounds

### Issue: Build Fails with "secret.properties not found"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [usetrmnl/trmnl-android](https://github.com/usetrmnl/trmnl-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
