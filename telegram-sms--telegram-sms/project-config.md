---
trigger: always_on
description: This file provides guidance to AI coding agents (Claude Code, Cursor, Codex, Copilot, Gemini, etc.) when working with code in this repository. Contributions developed with any AI tool are welcome — keep this file tool-neutral.
---

# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Cursor, Codex, Copilot, Gemini, etc.) when working with code in this repository. Contributions developed with any AI tool are welcome — keep this file tool-neutral.

## Project

Android app that forwards SMS / missed calls / battery events to a Telegram bot and accepts remote commands back. Single Gradle module `:app`, all Kotlin, target package `com.qwe7002.telegram_sms`.

- compileSdk / targetSdk: 36, minSdk: 23, JDK 21, Android Gradle Plugin 9.0.0
- No native code despite NDK abiFilters (`armeabi-v7a`, `arm64-v8a`) — those exist for transitive `.so` packaging (libsodium, conscrypt, MMKV)

## Build & test commands

The Gradle wrapper is at the repo root. On Windows use `gradlew.bat`; on POSIX runners use `./gradlew`.

```bash
./gradlew assembleDebug              # debug APK -> app/build/outputs/apk/debug/
./gradlew assembleRelease            # release APK (requires app/keys.jks + KEYSTORE_PASS/ALIAS_NAME/ALIAS_PASS env or -P props)
./gradlew test                       # JUnit unit tests
./gradlew :app:testDebugUnitTest --tests "com.qwe7002.telegram_sms.static_class.PhoneTest"
./gradlew clean
```

Version is **not** in `build.gradle.kts` — `versionCode` and `versionName` come from `VERSION_CODE` / `VERSION_NAME` env vars (set by GitLab CI, default `1` / `"Debug"` locally). Release tag scheme is Ubuntu-style `YY.MM[.N]` on `master`, timestamp on `nightly`. See [.reallsys/.gitlab-ci.yml](.reallsys/.gitlab-ci.yml).

Branch-based variant flavoring (also from `build.gradle.kts`):
- `nightly` branch → release variant gets `applicationIdSuffix=".nightly"` (parallel-installable)
- debug always gets `applicationIdSuffix=".debug"`

## Language pack (CRITICAL before any string-resource work)

`app/language_pack/` is a **git submodule** (`https://github.com/telegram-sms/language_pack.git`). It contains all `values-<locale>/` directories for non-English locales. The release CI runs `./gradlew app:copy_language_pack` which copies the submodule into `app/src/main/res/` before `assembleRelease`.

When working locally:

```bash
git submodule update --init --recursive    # first checkout
./gradlew app:copy_language_pack           # stage translations into res/
./gradlew app:clean_language_pack          # remove the staged values-* dirs again
```

Add new UI strings to `app/src/main/res/values/strings_*.xml` (category files — `strings_sms.xml`, `strings_telegram.xml`, `strings_battery.xml`, `strings_call.xml`, `strings_cc.xml`, `strings_chat.xml`, `strings_network.xml`, `strings_notification.xml`, `strings_privacy_about.xml`, `strings_scanner.xml`, `strings_sms_manage.xml`, `strings_update.xml`, `strings_ussd.xml`). Translations of those keys go into the **language_pack submodule**, not into `app/src/main/res/values-*` directly (those staged copies are gitignored / get clobbered by `copy_language_pack`).

**Auto-align translations after any string change.** Whenever you add, modify, or remove a UI string key in `app/src/main/res/values/strings_*.xml`, immediately bring every locale in the `language_pack` submodule into sync in the same change: add the new key to each `values-<locale>/` file (translated), update the text where the meaning changed, and delete keys you removed. Don't leave the source `values/` strings and the translations out of step. Edit the translations inside the `language_pack` submodule, never the staged `values-*` copies in `res/` (those get clobbered by `copy_language_pack`).

Two other submodules exist and are vendored into the main source tree (no copy step):
- `app/src/main/java/com/github/sumimakito/awesomeqrcode` — AwesomeQrRenderer
- `app/src/main/java/com/github/sumimakito/codeauxlib` — CodeauxLibPortable

## Architecture

### Process model

`MainApplication.onCreate()` only does `MMKV.initialize(this)`. The app is mostly **receivers + foreground services**, not Activity-driven:

- **Receivers**: `SMSReceiver`, `WAPReceiver` (MMS WAP push), `CallReceiver`, `BootReceiver`, `SMSSendResultReceiver`, `USSDCallBack`.
- **Foreground services** (`foregroundServiceType="specialUse"`):
  - `ChatService` — long-poll `getUpdates` against Telegram Bot API; routes commands to handlers in [static_class/ChatCommand.kt](app/src/main/java/com/qwe7002/telegram_sms/static_class/ChatCommand.kt). Holds a `WakeLock` and `WifiLock`.
  - `BatteryService` — battery + respond-via-message handler.
  - `NotificationService` — `NotificationListenerService` for app-notification → Telegram forwarding (Carbon Copy source).
- **JobServices**: `ReSendJob` (retry failed SMS sends), `CcSendJob` (Carbon Copy delivery), `KeepAliveJob`.
- **Activities** are config UI (`MainActivity`, `CcActivity`, `TemplateActivity`, `SpamActivity`, `ScannerActivity`, `TransferConfigActivity`, `LogActivity`, `NotifyActivity`).

`ChatService` is the brain — it owns the Telegram polling loop and dispatches inbound commands; outbound notifications come from receivers / `BatteryService` / `NotificationService` calling into `static_class.TelegramApi`.

### Package layout


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [telegram-sms/telegram-sms](https://github.com/telegram-sms/telegram-sms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
