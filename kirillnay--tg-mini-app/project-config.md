---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`tg-mini-app` (Maven: `io.github.kirillNay:tg-mini-app`) is a Kotlin Multiplatform library for building Telegram Mini Apps with Compose Multiplatform. It targets the browser only, with two targets, **`js` and `wasmJs`**, that share one source set, `src/webMain`. Kotlin and Compose versions are pinned in `gradle.properties` (`kotlin.version`, `compose.version`) and read by `settings.gradle.kts`. The wrapper covers the Telegram WebApp API up to Bot API 10.1.

## Commands

Library (from repo root):

```bash
./gradlew build                      # compile js + wasmJs and run browser tests
./gradlew jsBrowserTest              # tests on the js target only
./gradlew wasmJsBrowserTest          # tests on the wasmJs target only
./gradlew publishToMavenLocal        # try the library in another local project
```

Tests live in `src/webTest` and run in headless Chrome through Karma, once per target. To run a single test, add `--tests "com.kirillNay.telegram.miniapp.WebAppTest.<name>"`. There are three kinds:
- **`WebAppCallsTest`** uses `RecordingMock`, a JS Proxy that records every call as `Path.method([json args])` and can answer callbacks (`respondWith`). Every public method should have an assertion here.
- **`WebAppTest`** covers parsing and state on the hand-written `window.Telegram.WebApp` mock in `TelegramMock.kt`.
- **`TelegramWebAppContentTest`** is a set of Compose UI tests of `TelegramWebAppContent`. Use `composeUiTest { }` instead of `runComposeUiTest`: on js it waits for Skiko to load.

`bash scripts/ci/verify.sh` runs everything CI runs: the library build, the sample builds and the Playwright E2E.

If a build fails with "Lock file was changed", run `./gradlew kotlinUpgradeYarnLock`. Maven Central sometimes answers 403 from this machine; rerunning the build fixes it.

Sample app (`samples/coffee-order-demo`, a separate Gradle build; run from repo root):

```bash
./gradlew -p samples/coffee-order-demo :composeApp:wasmJsBrowserDevelopmentRun
./gradlew -p samples/coffee-order-demo :composeApp:composeCompatibilityBrowserDistribution   # what CI deploys
./gradlew -p samples/coffee-order-demo :androidApp:assembleDebug
./gradlew -p samples/coffee-order-demo :composeApp:linkDebugFrameworkIosSimulatorArm64
```

The sample's `settings.gradle.kts` does `includeBuild("../..")`, so its `io.github.kirillNay:tg-mini-app` dependency is **substituted with the local library source**.

Playwright E2E tests of the web sample live in `samples/coffee-order-demo/e2e`. Build the dists first (`:composeApp:composeCompatibilityBrowserDistribution :composeApp:jsBrowserDistribution`), then run `npm ci && npx playwright test`. Each scenario runs against both the wasm and js bundles.
- **Launch:** `openInTelegram` passes launch parameters in the URL hash and installs a `TelegramWebviewProxy` that records every `postEvent` of telegram-web-app.js.
- **Assertions:** tests assert on those posted events and simulate the client with `Telegram.WebView.receiveEvent`.
- **Clicks:** UI elements are found through Compose's accessibility tree (`getByRole`). Click them with `tap()`, which sends a real pointer event: the canvas intercepts Playwright's normal clicks.

To check the web sample by hand in a Mini App context without Telegram:
1. Serve `build/dist/composeWebCompatibility/productionExecutable`.
2. Open it with a URL hash of `tgWebAppData=...&tgWebAppVersion=8.0&tgWebAppThemeParams=...`, then reload. `telegram-web-app.js` reads its launch parameters from the hash.
3. To fire client events, call `Telegram.WebView.receiveEvent('back_button_pressed')` or `receiveEvent('theme_changed', {...})`.

## Library architecture

Packages live under `src/webMain/kotlin/com/kirillNay/telegram/miniapp/`.

**`webApp/internal`** holds the raw interop and is the only place that touches JS. Kotlin/Wasm has no `dynamic` and limits `js()` calls, so all interop goes through a few patterns:
- **`Externals.kt`**: `internal external interface …Js : JsAny`, with member names that match telegram-web-app.js exactly (snake_case included).
- **`Interop.kt`**: top-level `js("...")` helpers (`jsGet`, `jsTypeOf`, `jsFunction0/1`) and the `jsObject { put(...) }` builder, which skips `null` values.
- **`Callbacks.kt`**: `awaitResult` / `awaitValue` turn JS callbacks into suspend calls.
- **`CallbackRegistry`**: keeps the JS function created for each Kotlin lambda so that `offClick(lambda)` removes the same function `onClick` added.

Pitfalls here:
- The helpers are named `asStringOrNull` / `asDoubleOrNull` and so on, never `toXxxOrNull`. On the js target `JsString` *is* `String`, so a `toDoubleOrNull` extension on `JsAny?` shadows the stdlib one and recurses forever.
- Declare optional JS booleans as `JsAny?` and read them with `asBooleanOrNull()`. On wasm an undefined `Boolean?` external property comes back as `false`, not `null`.
- `initDataUnsafe` fields that come from the query string (`auth_date`, `can_send_after`) are strings at runtime. JSON fields such as `user.id` are numbers.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kirillNay/tg-mini-app](https://github.com/kirillNay/tg-mini-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
