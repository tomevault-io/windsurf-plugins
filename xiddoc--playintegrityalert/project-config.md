---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

PIAlert is an Xposed / LSPosed module that **notifies you the moment an app asks
for a Play Integrity verdict**. It hooks the Play Store (Finsky) process only,
reads the requesting app's package out of the integrity-request `Bundle`, and —
if that package is on the user's watch-list — fires a notification and records
the detection. It only **observes**; it never alters the verdict.

## Commands

Standard Gradle Android build (JDK 17 + Android SDK; `compileSdk`/`targetSdk` 35,
`minSdk` 30 — Android 11):

```bash
./gradlew :app:assembleDebug                       # debug APK → app/build/outputs/apk/debug/
./gradlew :app:assembleRelease                     # R8-minified, signed release APK → app/build/outputs/apk/release/
./gradlew :app:lintDebug                            # Android lint
./gradlew :app:testDebugUnitTest                    # run the JVM unit suite
./gradlew :app:jacocoTestReport                     # HTML coverage → app/build/reports/jacoco/
./gradlew :app:jacocoTestCoverageVerification       # enforce 100% line + branch (the gate)
./gradlew check                                      # runs tests + the coverage gate
```

Run a single test class or method:

```bash
./gradlew :app:testDebugUnitTest --tests "com.xiddoc.playintegrityalert.IntegrityRequestInspectorTest"
./gradlew :app:testDebugUnitTest --tests "*.AlertThrottleTest.debounces*"
```

## Coverage gate — non-negotiable

The build enforces **100% line and branch coverage of every hand-written class**;
only generated code (`R`, `BuildConfig`, `Manifest`) is excluded (see
`coverageExclusions` in `app/build.gradle.kts`). Any new or changed code must come
with tests that keep both counters at 1.0, or the build fails. This covers the
Xposed hook wiring and the UI Activities too — not just the pure logic.

The unit suite runs entirely on the host JVM (JUnit + Robolectric + MockK — no
device or emulator). To make the hook code run off-device, the suite ships
**functional fakes of the Xposed API** under `app/src/test/java/de/robv/...` and
`android.app.AndroidAppHelper`. These replace the published
`de.robv.android.xposed:api` jar, which is `compileOnly` in main and deliberately
kept *off* the test classpath (its real method bodies throw). When you add a hook
path that touches a new Xposed API, you must extend these fakes accordingly.

## Architecture — why it's shaped this way

The Play Integrity client libraries don't compute a verdict in-process; they hand
the request to **Google Play Store** (`com.android.vending`, "Finsky"), and the
caller's package travels *inside* that request. So the module injects into the
**Play Store process only** and watches the Finsky integrity services
(`Constants.INTEGRITY_SERVICE_CLASSES`). Hooking one process instead of every app
is far lighter, and because the caller package is in the request, that one process
still sees every app's request.

Runtime data flow (request → notification):

```
XposedEntry (loads only in com.android.vending)
  └─ IntegrityServiceHook        hooks the Finsky integrity service methods
       └─ IntegrityRequestInspector   PURE: is this a request? whose package?
            └─ WatchList (XSharedConfigSource)   is that package watched?
                 └─ AlertThrottle    per-caller debounce
                      └─ Notifier    explicit broadcast → our app's process
                           └─ DetectionReceiver   raises notification + DetectionStore
```

Key design seams to respect:

- **`IntegrityRequestInspector` is pure** — no Xposed dependency. The
  security-critical heuristics (request-vs-response discrimination via `token`/
  `error` keys, and caller extraction via `Constants.CALLER_PACKAGE_KEYS` plus a
  package-shaped-string fallback) live here precisely so they're fully JVM-testable.
  Keep new detection logic here, not in the hook.
- **Two caller-attribution paths.** The Bundle heuristics above cover the *classic*
  Play Integrity API, where the caller package rides inside the request `Bundle`.
  The *Standard/Express* API instead hands Finsky a Parcelable with no package, so
  `IntegrityServiceHook.callerFromBinder` falls back to the binder calling UID
  (`Binder.getCallingUid()`, always valid inside the transaction) and resolves it to
  a package via the Play Store process's `PackageManager` — filtering out system,
  Play Store, and our own UID. Without this, Standard-API requests are invisible.
- **Hook the returned binder, not just the service class.** The integrity *request*
  (`requestIntegrityToken` and friends) isn't a method on the `IntegrityService`
  class — Finsky returns an AIDL binder stub from `onBind` and dispatches the
  cross-process request onto *that* object. Hooking only the service's declared
  methods catches lifecycle/bind calls but never the request. So
  `IntegrityServiceHook.hookBinderResult` runs in the callback's `afterHookedMethod`:
  whenever a hooked service method returns an `IBinder`, it hooks that stub's methods
  too (once per class), which is where the request — and its caller UID — lands.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Xiddoc/PlayIntegrityAlert](https://github.com/Xiddoc/PlayIntegrityAlert) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-05 -->
