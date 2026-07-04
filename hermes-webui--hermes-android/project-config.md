---
trigger: always_on
description: This file is the shared entry point for AI assistants working in this
---

# Agent instructions for Hermes-Android

This file is the shared entry point for AI assistants working in this
repository. Keep it project-specific and safe to publish. Do not put private
machine setup, credentials, tokens, local network secrets, or personal workflow
notes here.

## Read first

Before making changes, read:

1. `README.md`
2. `ROADMAP.md`
3. `ARCHITECTURE.md`
4. `AGENTS.md`

For implementation work, also inspect the relevant source files under
`app/src/main/java/com/hermeswebui/android/`.

Useful entry points:

- `MainActivity.kt` - Android platform boundary, WebView, intents, downloads, dashboard Custom Tab launch
- `core/security/UrlPolicy.kt` - URL and navigation decisions; also contains the top-level `UrlOrigins` object with origin/URL normalization utilities (`hostFrom`, `hasSameOrigin`, `documentStartOriginRule`, `normalizeOriginUrl`, `normalizedPath`) — use these helpers rather than ad-hoc URI parsing
- `data/SettingsRepository.kt` - encrypted settings persistence; implements `SettingsStore` interface. Uses a versioned `runMigration()` pattern (`KEY_LAST_MIGRATION_VERSION`): when adding new data schema changes, increment `currentMigrationVersion` and add a corresponding migration block. Non-interface methods (`hasRequestedNotificationPermission`, `markNotificationPermissionRequested`, `getLastLoadedUrl`) are called directly by `MainActivity`.
- `domain/ServerUrlValidator.kt` - server URL validation rules
- `domain/ShareIntentParser.kt` - Android share-sheet parsing
- `ui/MainViewModel.kt` - app state orchestration
- `ui/web/WebShell.kt` - Compose WebView host and refresh/error UX

Known Android WebView compatibility behavior lives in `MainActivity.kt`:

- The Compose root applies `WindowInsets.safeDrawing` so the WebView shell and native snackbar do not overlap the Android status or navigation bars.
- Forced/algorithmic WebView darkening is disabled so Hermes WebUI keeps its own colors.
- WebView uses default browser-managed HTTP/service-worker caching and DOM storage for Hermes WebUI assets. Do not add a parallel native stale-site mirror for authenticated WebUI HTML/API responses; reset-session behavior must keep clearing cookies, WebStorage, and WebView cache.
- A measured viewport-height shim is injected because some Android WebView builds compute Hermes WebUI `100dvh` root layout height as `0px`, which hides page text/content.
- Android no longer writes WebUI `/api/dashboard/config`. WebUI owns the Official Hermes Dashboard setting, including Auto-detect and persistence. Android may normalize an explicitly configured local dashboard URL to its origin for Custom Tab matching and does not persist dashboard-origin pages as the app startup URL. OAuth/OIDC callback URLs for the configured Hermes WebUI origin must bypass dashboard Custom Tab matching and return to the primary WebView.
- WebView microphone access is handled in `MainActivity.kt` through Android `RECORD_AUDIO`, `MODIFY_AUDIO_SETTINGS`, plus `WebChromeClient.onPermissionRequest`; grant only `PermissionRequest.RESOURCE_AUDIO_CAPTURE` for trusted Hermes pages (prefer explicit allowlisted HTTP/HTTPS origins, with null/opaque-origin fallback only while the active main-frame URL is the configured Hermes WebUI route).
- Android WebView can expose Web Speech API objects that fail with `not-allowed` before the WebView permission bridge is used. Keep the document-start `mic_force_mediarecorder` fallback scoped to the configured Hermes WebUI origin so WebUI voice input uses MediaRecorder/getUserMedia instead.
- The injected Hermes WebUI "Application Settings" entry should anchor immediately after the WebUI Settings item when present, with Help only as a fallback anchor for older or changed sidebar markup. Do not add a persistent native overlay button for this entry; it should appear only with the WebUI sidebar.
- `hermes://app/settings` is exported as a native recovery deep link and opens `SettingsScreen` without relying on the current WebView route.
- WebUI browser notifications are handled in `MainActivity.kt` through Android `POST_NOTIFICATIONS`, a native notification channel, a document-start `Notification`/`ServiceWorkerRegistration.showNotification` compatibility facade, and an AndroidX WebMessageListener. Keep the bridge scoped to the configured Hermes WebUI route, reject subframes/non-WebUI origins, and validate notification tap URLs through the host allowlist before loading.
- Native app update alerts share the existing `Hermes updates` notification channel but are selected by build channel. Automatic checks should stay delayed until the app has been foregrounded for about one minute and throttled to once per day, while manual Settings checks run immediately. Keep the shared settings/notification UX common, with `BuildConfig.UPDATE_CHANNEL = "play"` using Google Play Core in-app updates, `"github"` checking GitHub Releases plus the `*-github.apk` asset for direct downloads and release-note excerpts, and `"none"` avoiding production update prompts in debug builds.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hermes-webui/hermes-android](https://github.com/hermes-webui/hermes-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
