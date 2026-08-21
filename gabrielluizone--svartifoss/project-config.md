---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Svartifoss — an Android app that lets a paired Wear OS watch control music playback on the phone (and on the watch itself), with customizable physical buttons / digital crown / gestures. **Sideload-only**: the phone and watch APKs are installed manually from GitHub releases (watch install via Wear Installer). A self-hosted F-Droid repo used to be served from GitHub Pages under `docs/fdroid/`; it was retired in 3.1-beta2 having fallen three versions behind, and GitHub releases are now the single distribution channel — don't reintroduce it without being asked. `fastlane/` at the repo root still holds store-listing metadata (descriptions, screenshots) in the Fastlane layout, now only of use to a future store attempt. It is *not* on the Play Store — a split free-phone-app / paid-watch-app listing was drafted (see `docs/play-console-*.md`) but the author shelved it, not a store enforcement action. A future Play Store attempt is more likely to be a single listing with in-app premium unlocks (Play Billing) than the old split-app model. Both APKs must still share the same `applicationId` and signing key (see "Package naming gotchas" below).

## Module layout

This is a multi-module Gradle (Groovy DSL) Android project, configured in `settings.gradle` / `build.gradle` / `libs.toml` (version catalog):

- `mobile/` — the phone app. Reads the currently playing media session, executes actions (play/pause/skip/volume/Tasker tasks/open other apps), and syncs config/state to the watch over the Wearable Data Layer API. Uses **Dagger 2** (`di/AppComponent.kt`, manual `dagger-android` injection) for DI.
- `wear/` — the watch app. Renders the on-watch UI (now-playing screen, action menu, volume bar), receives button/crown/gesture input, and talks to the phone. Uses **Hilt** for DI (different DI framework than `mobile/` — don't assume Dagger conventions carry over).
- `common/` — shared code linked into both `mobile` and `wear`: communication path constants (`CommPaths.kt`), action/button-config models, protobuf schemas (`src/main/proto/*.proto` — actions, music, watch, notifications, custom lists), shared views/drawables.
- `wearutils/` — a **git submodule** (https://github.com/gabrielluizone/WearUtils, this repo's fork of matejdro/WearUtils) with its own `libs.toml`. If this directory is empty, run `git submodule update --init` before building — Android Studio/Gradle sync will fail otherwise.

Root directories that are *not* part of the build: `icons/` (tracked; raw Material Symbols SVG downloads used as source material when adding drawables), `archived/` (parked scraps), `licenses/` (tracked; per-font license texts for the bundled typefaces), `wearmediatemplate/` (Google's Wear OS media-controls design guidance — screenshots plus `media-expressive.md` — kept as reference material for face styling), and the gitignored local reference folders `retromusic/`, `echo/` (a full local checkout of the Echo Music source — one of the three players `docs/player-integration-notes.md` documents from source, so read it there rather than guessing) and `redesign/`. Several stray build outputs / scratch files also sit at the repo root and are **not** build inputs (`full.apk`, `part.apk`, `mobile-debug.apk`, a bundled `com.google.android.wearable.media.sessions_*.apk`, `svartifoss_config.json`, `run.md`, `promp.txt`, `CHANGELOG.pdf`, `RELEASE_3.md` (a drafted GitHub release announcement, not shipping docs), assorted screenshot `*.png` scratch files, the gitignored `.playwright-mcp/` capture dir) — ignore them.

### Phone ⟷ watch communication

All communication goes through the Google Play Services **Wearable Data Layer API** (`MessageClient`/`DataClient`), with paths centralized in `common/.../CommPaths.kt`. Key entry points:
- Phone side: `WatchListenerService` (`WearableListenerService`) receives messages and forwards to `MusicService`; `WatchInfoProvider`/`ButtonConfigTransmitter`/`ActionListTransmitter` push config to the watch.
- Watch side: `PhoneConnection`, `WatchMusicService`, `PreferencesReceiver`, `IdleMessageListener` handle the corresponding receiving/sending logic.
- Communication is bidirectional. The watch reports *back* to the phone with `WatchInfoSender` (`watch/communication/WatchInfoSender.kt`), which sends the watch's own button hardware capabilities as a `WatchInfo` (from `watch.proto`) so the phone knows which physical buttons exist. (Note the phone-side `WatchInfoProvider` is a different thing — it *pushes* config to the watch.)
- Payloads for structured data (action lists, button configs, watch info) are serialized with **protobuf** using the schemas in `common/src/main/proto/`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gabrielluizone/Svartifoss](https://github.com/gabrielluizone/Svartifoss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
