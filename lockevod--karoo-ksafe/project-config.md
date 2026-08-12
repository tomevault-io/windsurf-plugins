---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

KSafe is a free, open-source safety extension for Hammerhead Karoo cycling computers (Karoo 3, Karoo OS 1.527+). Single-module Android Gradle project (`:app`) written in Kotlin + Jetpack Compose. The APK is both a settings UI (`MainActivity`) and a Karoo extension service (`KSafeExtension`) registered via the Hammerhead Karoo SDK. There is no separate library module.

## Toolchain

- AGP 8.5.0, Kotlin 2.0.0, Compose compiler plugin, kotlinx.serialization plugin.
- JDK 17 (`sourceCompatibility`/`kotlinOptions.jvmTarget = 17`).
- `compileSdk = 34`, `targetSdk = 34`, `minSdk = 23`.
- Sources live under `app/src/main/kotlin/...` (note: `kotlin/`, not `java/`). Resources under `app/src/main/res/`.
- The Hammerhead Karoo SDK (`io.hammerhead:karoo-ext`) is pulled from a private Maven repo (`maven.pkg.github.com/hammerheadnav/karoo-ext`). `settings.gradle.kts` reads `gpr.user`/`gpr.key` from `local.properties` and **fails the build hard** (`error("File from not found")`) if `local.properties` is missing — even for a clean checkout. A valid `local.properties` with `sdk.dir`, `gpr.user`, `gpr.key` is therefore required to do anything with Gradle.
- `app/build.gradle.kts` also reads `calib.bot_token` and `calib.chat_id` from `local.properties` and exposes them as `BuildConfig` fields. Empty values are fine — `LogReporter` then no-ops the calibration log upload. `local.properties` is in `.gitignore`; never commit it.

## Common commands

```bash
# Build (debug + release APKs land in app/build/outputs/apk/...)
./gradlew :app:assembleDebug
./gradlew :app:assembleRelease

# Lint / static analysis (Android Lint only — no detekt/ktlint/spotless configured)
./gradlew :app:lint

# Install onto a connected device or emulator
./gradlew :app:installDebug

# Clean
./gradlew clean
```

JVM unit tests live under `app/src/test/kotlin/...` and run via `./gradlew :app:testDebugUnitTest`. The suite uses JUnit 4 + `kotlinx-coroutines-test` + Mockito (testImplementation dependencies in `libs.versions.toml`). There is no `androidTest` instrumentation source set — tests must be JVM-runnable. To run a focused subset: `./gradlew :app:testDebugUnitTest --tests "com.enderthor.kSafe.extension.crash.*"`.

The release build is configured to sign with the **debug** signing config and runs R8 minification (`isMinifyEnabled = true`) — keep ProGuard rules in `app/proguard-rules.pro` in mind when adding reflection-based code.

## High-level architecture

The whole app revolves around the Karoo extension service. Internet, GPS, ride state, sensor streams, and HTTP all go through the `KarooSystemService` provided by `karoo-ext` — there are no direct OkHttp/Retrofit calls and no GPS code outside `LocationManager`.

### Service layer (`extension/`)

- **`KSafeExtension`** (`extension/KSafeExtension.kt`) — the central orchestrator. Subclasses `io.hammerhead.karooext.extension.KarooExtension`, exposes the `DataType`s, owns the `KarooSystemService`, instantiates every manager, and implements `onBonusAction` for hardware buttons. All long-running work runs on `Dispatchers.Main + SupervisorJob` belonging to this service. A static `getInstance()` lets `DataType` callbacks reach back into the service when they cannot be passed dependencies through Karoo's callback API.
- **`Sender`** (`extension/Sender.kt`) — sends messages via the four supported providers (CallMeBot/WhatsApp, Pushover, ntfy, Telegram). Three entry points: `sendAlert` (emergency, retries with 3 cycles × 3 attempts × 60/120/180s backoff), `sendInfo` (single attempt, used for ride start/end and custom messages), `testSend` (single attempt, returns a human-readable string for the Settings UI).
- **`Extensions.kt`** — `KarooSystemService` extension functions that wrap the SDK's callback API into `Flow`s (`streamRide`, `streamDataFlow`, `streamRideProfile`, `httpRequest`, …). Anywhere you need a Karoo data stream as a Flow, add it here rather than reimplementing the callback boilerplate.

### Managers (`extension/managers/`)

- **`ConfigurationManager`** — DataStore Preferences wrapper. Stores three JSON blobs under string keys (`ksafeconfig`, `sender`, `emergencystate`). Reads run through `migrateToLatest()` (see `data/ConfigData.kt` — `CONFIG_VERSION` is bumped when defaults change) and through inline string substitutions for renamed enums (e.g. `SIMPLEPUSH` → `NTFY`). Use `jsonWithUnknownKeys` for reads and `jsonForExport` for user-visible exports.
- **`CrashDetectionManager`** — Android `SensorManager` listener (accelerometer + gyroscope, `SENSOR_DELAY_GAME` ≈ 50 Hz). Implements the multi-stage state machine described in `docs/crash-detection-algorithm.md`: `MONITORING → IMPACT → SILENCE_CHECK → CRASH_CONFIRMED`, with a parallel speed-drop monitor and a dual smoothed/peak detector. Reads speed/cadence/grade/ride-profile from streams pushed in by `KSafeExtension`. **Do not change the sensor type to `TYPE_LINEAR_ACCELERATION`** — the stillness logic compares magnitude against gravity (~9.81 m/s²) and silently breaks otherwise.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lockevod/Karoo-KSafe](https://github.com/lockevod/Karoo-KSafe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
