---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Flutter app template for a **single-forum** XenForo client. The app is hard-bound to one forum at build time via `lib/config/app_forum_config.dart` (no runtime forum chooser, no `forumcopilot.com` runtime APIs). It talks to the forum through a server-side XenForo add-on (the `forumcopilot.php` endpoint) shipped under `plugins/FC_XenForo2/`.

Flutter `^3.6.1` / Dart `^3.6.1`. Targets Android, iOS, macOS, Windows, Linux, web.

## Repository layout (the parts that matter)

- `lib/` — the app. Entry point `lib/main.dart` → `ForumCopilotApp` in `lib/forumcopilot_app.dart` → home `views/single_forum_bootstrap_page.dart`.
- `lib/config/app_forum_config.dart` — **the only file a fork normally edits**: forum name, base URL, plugin endpoint, push mode, Android passkey identifiers. `AppForumConfig.buildSite()` constructs the singleton `Site` used throughout.
- `lib/controllers/` — GetX controllers (`SiteController`, `SiteManager`, `LoginController`, `TopicController`, `PostController`, `PushNotificationController`, `NotificationSettingsController`, `GlobalLoaderController`).
- `lib/services/` — initialization, push, user state, site proxy wiring, notification prefs.
- `lib/views/` — pages and widgets (forum list, topics, threads, posting, PMs, settings, profile, search). Posting flows include attachments, BBCode, and polls.
- `lib/core/` — cross-cutting: `errors/` (with Crashlytics init), `logging/AppLogger`, `memory/MemoryManager`, `cache/`, `async/`.
- `lib/l10n/` — ARB files; generated output in `lib/l10n/generated/` via `flutter gen-l10n` (config in `l10n.yaml`).
- `packages/forumcopilot_sdk/` — local package. Forum-agnostic abstractions: `IFC*Proxy` interfaces, `FC*Result` response wrappers, `SiteContext`, `SiteProxyFactory`, networking (Dio with persistent cookies, Cloudflare hooks). Uses `dart_mappable` + `json_annotation` codegen.
- `packages/xenforo_core/` — local package. Concrete XenForo implementation of the SDK proxies (`XenForoProxyFactory` + per-area proxies + XenForo→FC converters). Uses the forum's REST API plus the Forum Copilot add-on.
- `plugins/FC_XenForo2/` — the server-side XenForo add-on (PHP). **This is the add-on's official source**; no other repo should carry a copy (a customer-specific fork exists and is not upstream). `plugins/FC_XenForo2/README.md` covers layout, building the ZIP and releasing. `deploy_plugin.sh` rsyncs it to a server. Not part of the Flutter build.
- `docs/guides/` — platform-specific setup notes (macOS file picker entitlements, splash, icons, reset).
- `docs/REPOSITORIES.md` — the four related repositories, the one local checkout to use for each, what each owns, and how app and add-on releases flow to forumcopilot.com. Read it before touching `packages/`, `plugins/`, or anything release-related.
- `docs/xenforo_api_docs/` — XenForo REST reference; **excluded from analysis** (`analysis_options.yaml`).

## Architecture in one paragraph

`main.dart` runs critical init (error handling, `MemoryManager`, `ForumcopilotSdk.ensureInitialized`, `UserStateService`, `SettingsContext.loadFromDevice`) then `runApp(ForumCopilotApp())`. Firebase + push init runs **in the background after `runApp`** so the UI does not block on it; `PushNotificationController` is created lazily once an FCM token arrives. `ForumCopilotApp` registers `GlobalLoaderController` and `SiteController`, then renders `GetMaterialApp` with a global loader overlay and `UserStateBanner`. The home is `SingleForumBootstrapPage`, which builds the forum's `Site` from `AppForumConfig` and drives the rest of the app. State is managed with **GetX** (`Get.put` / `Obx`); navigation uses `globalNavigatorKey` (defined in `forumcopilot_sdk`) so SDK code can show dialogs (e.g. Cloudflare challenge UI) without a `BuildContext`. All forum I/O goes through `SiteProxyFactory.get*Proxy()` returning the `xenforo_core` implementations registered at startup; results follow a uniform `FC*Result { result, resultText, ...payload }` shape.

## Common commands

Bootstrap a fresh checkout (run from repo root):

```bash
flutter pub get
./buildlib.sh          # codegen for forumcopilot_sdk + flutter gen-l10n
                       # Windows: buildlib.bat
```

`buildlib.sh` runs `dart run build_runner build --delete-conflicting-outputs` inside `packages/forumcopilot_sdk` and then `flutter gen-l10n`. **Re-run it whenever** you change ARB files, or any `dart_mappable` / `json_annotation` annotated class in the SDK. `xenforo_core` also has codegen — if you touch its annotated classes, run `dart run build_runner build --delete-conflicting-outputs` inside `packages/xenforo_core` as well.

Run / build:

```bash
flutter run -d macos                  # also: -d chrome, -d <ios-device>, -d <android-id>
flutter build macos                   # release; output under build/macos/Build/Products/Release/
flutter analyze
```

Tests:

```bash
flutter test                          # app-level (just test/widget_test.dart)
# Live API suite (writes to the forum; use a throwaway instance). Reads
# packages/xenforo_core/test/config.json (copy from config.json.example):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [forumcopilot/xenforoapp](https://github.com/forumcopilot/xenforoapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
