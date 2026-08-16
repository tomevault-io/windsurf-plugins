---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

This is the **Branch Android SDK** for deep linking and attribution. Production code is a single Android library module, `:Branch-SDK`, published to Maven Central as `io.branch.sdk.android:library`. Package root: `io.branch.referral`.

## Modules

- **`Branch-SDK`** — the published SDK library (`com.android.library`). All production code lives under `Branch-SDK/src/main/java/io/branch/`, most of it in `referral/`.
- **`Branch-SDK-TestBed`** — a sample/demo app (`com.android.application`, `io.branch.branchandroidtestbed`) that depends on `:Branch-SDK`. Used for manual testing and as the target app for E2E tests.
- **`Branch-SDK-GPTDriver`** — a `com.android.test` module (MobileBoost/GPTDriver hybrid E2E tests) whose `targetProjectPath` is `:Branch-SDK-TestBed`. Deterministic Espresso first, AI-assisted validation only when Espresso can't express the intent. Requires a `MOBILEBOOST_API_KEY` (see `local.properties.example` / `Branch-SDK-GPTDriver/README.md`).

## Source layout

Most code is in `io.branch.referral`, but not all — check these sibling packages under `Branch-SDK/src/main/java/io/branch/` before assuming a class lives in `referral/`:

- **`referral/`** — the SDK core: `Branch`, `PrefHelper`, `ServerRequest*` + `ServerRequestQueue`, device/tracking/config, link & share builders. Sub-dirs: `network/` (`BranchRemoteInterface` + `…UrlConnection` HTTP impl), `util/` (`BranchEvent`, `BRANCH_STANDARD_EVENT`, `LinkProperties`, `CommerceEvent`, content-metadata types), `validators/` (integration & deep-link diagnostics), `QRCode/`.
- **`indexing/`** — `BranchUniversalObject` (the BUO content model used for link creation and tracking).
- **`coroutines/`** — Kotlin coroutine entry points for async fetches: `AdvertisingIds`, `DeviceSignals`, `InstallReferrers`.
- **`data/`** — `InstallReferrerResult` (result model for the referrer fetch).
- **`interfaces/`** — public callback interfaces (e.g. `IBranchLoggingCallbacks`).
- **`receivers/`** — `SharingBroadcastReceiver` (captures the chosen app from the system share sheet).

## Where to make changes

| Task | Start here |
| --- | --- |
| Session/init flow, deep-link callbacks, intent parsing | `Branch.java` (`sessionBuilder`, `initializeSession`, `registerAppInit`, `readAndStripParam`) |
| A new API request type or changing request bodies | subclass `ServerRequest`; wire dispatch/gating in `ServerRequestQueue.java`; add path to `Defines.RequestPath` |
| Persisted state / new SharedPreferences key | `PrefHelper.java` (add `KEY_*` + typed accessors) |
| Wire-format field names / enums / endpoints | `Defines.java` (`Jsonkey`, `RequestPath`, `IntentKeys`, `HeaderKey`) |
| Device/hardware signals on requests | `DeviceInfo.java`, `SystemObserver.java`, `coroutines/DeviceSignals.kt` |
| Ad ID (GAID/Huawei/etc.) fetch | `coroutines/AdvertisingIds.kt` |
| Install-referrer fetch | `coroutines/InstallReferrers.kt`, `data/InstallReferrerResult.kt`, `AppStoreReferrer.java`, `BranchPreinstall.java` |
| Link creation (short/long URLs) | `BranchShortLinkBuilder.java`, `BranchUrlBuilder.java`, `BranchLinkData.java`; content model in `indexing/BranchUniversalObject.java`, `util/LinkProperties.java` |
| Sharing / share sheet | `BranchShareSheetBuilder.java`, `ShareLinkManager.java`, `NativeShareLinkManager.java`, `receivers/SharingBroadcastReceiver.kt` |
| Custom analytics / commerce events | `util/BranchEvent.java`, `util/BRANCH_STANDARD_EVENT.java`, `util/CommerceEvent.java` |
| Tracking-disabled / consent / DMA | `TrackingController.java`, plus the DMA/consent keys in `PrefHelper.java` |
| `branch.json` / config flags | `BranchJsonConfig.java`, `BranchConfigurationController.kt`, `BranchUtil.java` (key resolution) |
| HTTP transport / retries / timeouts | `network/BranchRemoteInterfaceUrlConnection.java` (impl), `network/BranchRemoteInterface.java` (abstract) |
| QR codes | `QRCode/BranchQRCode.java`, `QRCode/ServerRequestCreateQRCode.java` |
| Integration / deep-link diagnostics | `validators/IntegrationValidator.java`, `validators/DeepLinkRoutingValidator.java` |
| Logging | `BranchLogger.kt`, `interfaces/IBranchLoggingCallbacks.java` |

## Build, test, lint

Toolchain: **Java 17** for Gradle (CI uses Corretto/Temurin 17), `compileSdk 34`, `minSdk 21` for `:Branch-SDK` (`:Branch-SDK-GPTDriver` sets `minSdk 24`). Copy `local.properties.example` → `local.properties` (Android Studio fills in `sdk.dir`).

```bash
# JVM unit tests (fast, no emulator) — the everyday loop
./gradlew :Branch-SDK:testDebugUnitTest

# A single unit test class or method
./gradlew :Branch-SDK:testDebugUnitTest --tests "io.branch.referral.BranchConfigurationControllerTest"
./gradlew :Branch-SDK:testDebugUnitTest --tests "*BranchConfigurationControllerTest.someMethod"

# Instrumented tests (need a connected device/emulator) — the bulk of the suite
./gradlew :Branch-SDK:connectedDebugAndroidTest
# A single instrumented class
./gradlew :Branch-SDK:connectedDebugAndroidTest -Pandroid.testInstrumentationRunnerArguments.class=io.branch.referral.BranchApiTests


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BranchMetrics/android-branch-deep-linking-attribution](https://github.com/BranchMetrics/android-branch-deep-linking-attribution) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
