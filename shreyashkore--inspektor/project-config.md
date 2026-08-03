---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Inspektor is a Kotlin Multiplatform HTTP inspection library for Ktor (think Chucker, but multiplatform). It installs as a Ktor client plugin, persists every request/response to a local SQLite database, and ships its own Compose Multiplatform UI for browsing transactions. It also supports *overriding* outgoing requests / incoming responses for testing.

Targets: **Android, JVM (Desktop), and iOS** (iosArm64 + iosSimulatorArm64). The published artifact is `com.gyanoba.inspektor:inspektor`.

The repo has two modules: `:inspektor` (the library) and `:sample` (a Compose Multiplatform demo app). `iosApp/` is the Xcode project for the sample.

## Commands

Use the Gradle wrapper (`./gradlew`). The toolchain expects **JDK 17**; the Android target compiles to JVM 1.8 bytecode.

```bash
# Compile the library for a single target (fast feedback loop)
./gradlew :inspektor:compileKotlinJvm

# Run all library unit tests (CI runs only the JVM target)
./gradlew :inspektor:jvmTest

# Run a single test class / method (JVM)
./gradlew :inspektor:jvmTest --tests "RetentionManagerTest"
./gradlew :inspektor:jvmTest --tests "RetentionManagerTest.checkAndCleanUp should clean up after cleanupFrequency for short retention"

# Android lint
./gradlew lint

# Build the sample app (Android + desktop)
./gradlew :sample:assembleDebug
./gradlew :sample:run        # desktop (mainClass = "MainKt")
```

### Public API / binary compatibility (important)

The library uses `explicitApiWarning()` and the [binary-compatibility-validator](https://github.com/Kotlin/binary-compatibility-validator). Any change to the **public** API surface will fail `apiCheck`. If a public API change is intentional, regenerate the signature dump and commit it:

```bash
./gradlew apiCheck   # verify
./gradlew apiDump    # regenerate inspektor/api/*.api after intentional changes
```

Because of this, prefer `internal` visibility for anything not meant to be public. The `com.gyanoba.inspektor.data` package and the sample module are excluded from API validation (see `apiValidation {}` in the root `build.gradle.kts`).

## Architecture

### The plugin pipeline (`Inspektor.kt`)

`Inspektor` is a Ktor `ClientPlugin<InspektorConfig>` built with `createClientPlugin`. It hooks into custom interception points defined in `utils/KtorHooks.kt` (`SendStateHook`, `SendMonitoringHook`, `ReceiveStateHook`, `ResponseReceiveHook`) rather than the stock Ktor logging hooks, because it needs to *mutate* requests/responses for the override feature. The flow per call:

1. **SendStateHook** – decide whether to log (via `filter {}` predicates), create an `HttpClientCallLogger`, run retention cleanup, and apply any matching **request** override (swap body/headers, recording the originals).
2. **SendMonitoringHook** – capture request info/headers/body. The request body is read non-destructively by `OutgoingContent.observe()` (`utils/ObservingUtils.kt`), which tees the byte stream into a logging channel.
3. **ReceiveStateHook** – capture response info/headers, and apply any matching **response** override (replacing the response via `replaceResponse`).
4. **ResponseObserver** – read the response body (only installed when `level.body` is true).

`LogLevel` (NONE/INFO/HEADERS/BODY) gates how much is captured. A per-call `HttpClientCallLogger` (`HttpClientLogger.kt`) accumulates state into a `MutableHttpTransaction`, then writes it to the data source. It uses two `Job` monitors + atomic flags so the response log always waits for the request log to be inserted first (`closeResponseLog` joins `requestLoggedMonitor`), guaranteeing the row exists before it's updated.

### Data layer (`data/`)

- **SQLDelight** is the persistence engine. The schema and all queries live in `inspektor/src/commonMain/sqldelight/.../HttpTransaction.sq` (database name `InspektorDatabase`, package `com.gyanoba.inspektor.data`). Generated code is what `InspektorDataSourceImpl` calls. Header sets and `Instant` columns are stored via `ColumnAdapter`s in `data/adapters/` (JSON-encoded).
- `InspektorDataSource` / `InspektorDataSourceImpl` wraps the queries with coroutine `Dispatchers.IO`. `getTransaction` / `getTransactionFlow` return **nullable** types and use the `*OrNull` query variants.
- The SQL driver is created per-platform via `expect object DriverFactory` (`data/Db.kt`), with actuals in `data/Db.{android,jvm,apple}.kt`.
- **Overrides** (`data/Override.kt`, `data/OverrideRepository.kt`) are stored separately using **KStore** (a JSON file `overrideStore`), not SQLDelight. An `Override` has a `RequestType`, a list of `Matcher`s (`UrlMatcher`, `UrlRegexMatcher`, `HostMatcher`, `PathMatcher`), and an `OverrideAction` (FixedRequest / FixedResponse / FixedRequestResponse). Matching logic lives in `Matcher.matches(...)` in `Inspektor.kt`.

### UI (`ui/`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ShreyashKore/inspektor](https://github.com/ShreyashKore/inspektor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
