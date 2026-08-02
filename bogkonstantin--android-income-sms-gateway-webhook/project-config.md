---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A free, open-source Android app (`tech.bogomolov.incomingsmsgateway`) that forwards incoming SMS to a user-configured URL as an HTTP POST with a JSON body. No backend, no accounts. The repo is intentionally a "stable, minimal" build — it is maintained but not actively expanded, so prefer small, focused changes over large refactors.

## Build & test

The project uses Gradle (wrapper committed, Gradle 9.4.1) and the Android plugin (AGP 9.2.1, `compileSdkVersion 33`, `minSdkVersion 14`). Source is Java, not Kotlin. AGP 9.x requires JDK 17 or newer — build with JDK 17 (e.g. `JAVA_HOME=<jdk17> ./gradlew ...`).

```bash
./gradlew assembleDebug        # build debug APK
./gradlew assembleRelease      # build release APK (minify disabled)
./gradlew testDebugUnitTest    # run the JVM unit tests (no device needed)
./gradlew connectedAndroidTest # run the instrumented tests (REQUIRES a connected device/emulator)
```

Most tests live in `app/src/androidTest/` and are **instrumented tests** — they run on a device/emulator, not the local JVM. There are also a few pure-JVM unit tests in `app/src/test/` (e.g. `ForwardingConfigPrepareMessageTest`) that run with `testDebugUnitTest`; prefer adding tests there when the logic under test touches no Android APIs. Run a single instrumented test class with:

```bash
./gradlew connectedAndroidTest -Pandroid.testInstrumentationRunnerArguments.class=tech.bogomolov.incomingsmsgateway.WebhookCallerTest
```

CI runs both suites via GitHub Actions (`.github/workflows/tests.yml`): a fast JVM unit-test job, and an emulator job for the instrumented tests.

When adding a feature, cover it with tests at the same level the codebase already uses: a `SharedPreferences`-backed model gets an instrumented model test (see `FailedMessageTest`, `HeartbeatSettingsTest`), and a screen's form validation gets an Espresso instrumented test driving the views (see `MainActivityTest`, `SettingsActivityTest`). For an Espresso test, don't exercise paths that start the foreground service (e.g. a valid "save" that calls `startForegroundService`) — it leaves a real service/notification running; cover the persistence separately via the model test instead.

`local.properties` is **git-ignored** (not committed); it points to the Android SDK locally, and CI falls back to the runner's `ANDROID_HOME`.

## Architecture

The data flow is: **SMS arrives → matched against configs → a WorkManager job POSTs to the webhook with retries.**

- **`SmsReceiverService`** — a foreground `Service` (the "F" status-bar icon) that keeps the process alive and hosts the heartbeat ping. Since issue #78 it registers **no** SMS receiver: `SmsBroadcastReceiver` is declared in the manifest (`SMS_RECEIVED` is on the implicit-broadcast allowlist, and the entry is guarded by the system-only `BROADCAST_SMS` permission), so the system cold-starts the process to deliver an SMS even after an aggressive OEM battery manager kills the service. Don't add a runtime `registerReceiver` back — that would double-deliver every message. Started by `MainActivity` when at least one config exists, and re-started on device boot by `BootCompletedReceiver`.

- **`SmsBroadcastReceiver.onReceive`** — the core dispatch logic (manifest-declared, see above). Reconstructs the message from PDUs, loads all `ForwardingConfig`s, and for each config checks: sender matches (exact `String.equals`, the asterisk wildcard `*`, or — when the per-rule "sender is a regular expression" flag is on (issue #88) — a regex `find()` that *fails closed* on an invalid pattern), SMS forwarding is enabled, the optional per-rule text filter regex matches the body (issue #52; empty forwards everything, an invalid pattern *fails open*), and SIM slot matches. SIM slot detection is heuristic (`detectSim`) because the bundle key for SIM slot is OEM-specific — it probes many known key names. Matching configs are dispatched via `callWebHook`, which enqueues a `OneTimeWorkRequest` with `NetworkType.CONNECTED` constraint and exponential backoff.

- **`RequestWorker`** (a WorkManager `Worker`) — runs the actual HTTP call off the main thread. Reads config from input `Data`, delegates to `Request`, and maps the result to `Result.success/retry/failure`. Retries stop once `getRunAttemptCount()` exceeds the config's max retries (default 10). WorkManager handles the backoff and waiting-for-network, which is how "retry with exponential backoff" is implemented. When the per-config "store failed messages" option is on, a request that exhausts its retries (or errors permanently) is persisted via `FailedMessage` instead of being silently dropped.

- **`FailedMessage`** — opt-in storage for messages that never delivered (issue #3). Persists the failed payloads in their own `SharedPreferences` file and exposes `getCount()` / `retryAll()` (re-enqueues them through `RequestWorker`). Surfaced in the UI by the "Retry N failed" action-bar item in `MainActivity`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bogkonstantin/android_income_sms_gateway_webhook](https://github.com/bogkonstantin/android_income_sms_gateway_webhook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
