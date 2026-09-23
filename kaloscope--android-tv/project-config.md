---
trigger: always_on
description: Native Android TV client for [kaloscope/kaloscope](https://github.com/kaloscope/kaloscope).
---

# Kaloscope Android TV Agent Guide

Native Android TV client for [kaloscope/kaloscope](https://github.com/kaloscope/kaloscope).
The server is user-managed and is not part of this repository.

## Working rules

1. Run `git status --short` before editing; preserve unrelated user changes.
2. Use `rg` to inspect the affected implementation, callers, resources, and
   tests. Start with the code map below rather than scanning the entire project.
3. For client behavior, follow the user's requirements, then current production
   code and tests. For build assumptions, read the Gradle files. For API changes,
   verify routes, encoding, DTOs, and behavior against public upstream source or
   documentation; local fixtures alone do not prove the server contract.
4. Implement the smallest complete change. Avoid unrelated refactors,
   speculative abstractions, empty screens, and placeholder repositories.
5. Run relevant targeted checks, inspect the final diff, and report what changed,
   what was verified, and any remaining uncertainty. Never invent API contracts.

Keep the project portable: do not make it depend on sibling repositories,
absolute paths, local accounts, browser sessions, private servers, or
machine-specific tools. Do not assume a server, device, emulator, or signing key
is available. Prototype content is visual reference only; production uses real
repositories. Fixtures and sample data belong only in tests or previews.

Never commit or log credentials, tokens, cookies, private server URLs, media
paths, keystores, or local configuration. Use synthetic data in tests and
examples. Do not log authorization headers or response bodies containing private
data, or inspect private configuration to obtain test credentials.

## Code map and architecture

Kotlin paths below are relative to `app/src/main/java/org/kaloscope/tv/`;
resource paths are relative to the repository root.

| Area | Start here |
| --- | --- |
| Startup and authentication | `app/KaloscopeApp.kt`, `app/KaloscopeViewModel.kt`, `app/bootstrap/`, `app/RootStateInspection.kt` |
| Shell and navigation | `app/MainShell.kt`, `app/MainShellActions.kt`, `app/navigation/MainNavigation.kt` |
| Server accounts and tokens | `feature/server/`, `data/server/`, `data/auth/`, `core/storage/` |
| Browsing and resource resolution | `feature/home/`, `feature/search/`, `feature/library/`, `feature/detail/`; `data/history/`, `data/media/`, `data/search/` |
| Playback | `core/player/`, `feature/player/` |
| Image and text reading | `core/reader/`, `feature/reader/`, `data/reader/` |
| Settings | `core/model/TvSettings.kt`, `feature/settings/`, `data/settings/` |
| HTTP and dependency injection | `core/network/KaloscopeApi.kt`, `core/network/ApiClientFactory.kt`, `core/network/NetworkCall.kt`, `app/di/AppModule.kt` |
| Shared UI | `core/designsystem/`, `app/KaloscopeTheme.kt`, `app/src/main/res/` |

Keep the package boundaries: `app` wires the shell, bootstrap, navigation, and
DI; `core` holds shared models, storage, networking, design system, and playback
and reading policies; `data` implements repositories, mapping, and persistence;
`feature` owns screen UI, ViewModels, and coordinators. Add feature DTOs under
`data/<area>/remote/`; shared envelopes and some existing API DTOs live in
`KaloscopeApi.kt`. Do not move those merely to satisfy a layering preference.
UI must not call Retrofit or DataStore directly. Communicate between features
through models, callbacks, routes, and IDs, not another screen's internals.

Established stack:

- One `app` module, one Activity, Kotlin, Java 17, minimum API 23, and
  application ID/namespace/root package `org.kaloscope.tv`.
- Jetpack Compose and TV Material, Navigation 3 with serializable route keys,
  Hilt, StateFlow, and testable coordinators for complex state transitions.
- Retrofit, OkHttp, Kotlinx Serialization, Preferences DataStore, Android
  Keystore token encryption, Coil, Media3 (ExoPlayer, MediaSession, HLS, DASH,
  Compose UI), and AkDanmaku.

Read `app/build.gradle.kts`, `gradle/libs.versions.toml`, `gradle.properties`, and
`gradle/wrapper/gradle-wrapper.properties` for current SDK, plugin, and dependency
versions. Use the wrapper and version catalog. The build uses AGP's built-in
Kotlin and `com.android.legacy-kapt`; preserve the documented `BuildConfig`,
`kotlin-metadata-jvm` kapt dependency, and in-process compiler workarounds unless
replacing them is in scope and the replacement is verified.

Do not introduce Leanback UI, Fragment/XML primary UI, Room, another HTTP or DI
framework, a service locator, multiple modules, an event bus, or WebView product
flows without explicit approval and a clear migration need. The manifest's
Leanback TV feature and launcher declarations are not the Leanback UI toolkit.

## Product and navigation invariants

- Server setup/login and the authenticated shell are mutually exclusive root
  states. Saved servers retain separate tokens. Authentication failure clears
  the affected session; ordinary network failures and forbidden access do not.
  Keep nested feature errors covered by `app/RootStateInspection.kt`.
- Home shows only `video` watch history. Network video and reader content must
  not create local `MediaItem` history.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kaloscope/android-tv](https://github.com/kaloscope/android-tv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
