---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**Gravity Music** (Flutter package name `saragama` — successor to the older "Saragama" app) — a Flutter music player that streams audio from YouTube via `youtube_explode_dart`. Playback architecture is a Flutter port of **HarmonyMusic**; several files say "mirrors HarmonyMusic's X" — when in doubt about *why* something is structured a certain way, that's the reference implementation it was ported from.

**Android is the primary target, but Linux/Windows desktop is actively supported** (media_kit audio backend + MPRIS system integration, see `main.dart`). iOS/web folders exist but are unconfirmed.

## Commands

- `flutter pub get` — install dependencies
- `flutter run` — run on a connected device/emulator (or `-d linux` / `-d windows` for desktop)
- `flutter analyze` — static analysis (uses `flutter_lints`, see `analysis_options.yaml`)
- `flutter test` — run the test suite (`test/` — pure-Dart service logic + a widget test; the audio handler is never booted)
- `flutter test test/services/taste_profile_test.dart` — run a single test file
- `flutter build apk` — build Android release APK. `android/gradle.properties` caps Gradle at `-Xmx4G` deliberately — the previous 8G+4G-metaspace setting got the JVM OOM-killed (build exit 137) on a 15 GB/no-swap machine; don't raise it back without reason.
- `flutter build windows` — Windows release runner (Windows only). CI additionally packages it into an Inno Setup `.exe` installer via `windows/packaging/gravity-music.iss` (see Platform integration).
- `dart run flutter_launcher_icons` — regenerate launcher icons (Android/iOS/**Windows `.ico`**) from `assets/app_icon.png`

## Releasing

Releases are cut by pushing a **`vX.Y.Z`** git tag on `main` — CI (`.github/workflows/build-packages.yml`) then builds the Android APK + Linux `.deb`/`.rpm` + Windows `.exe`/`.msix` and publishes a GitHub Release with them attached. The **in-app updater** (`UpdateService`, `services/update_service.dart`) reads that release, so **version consistency is load-bearing**:

- **`pubspec.yaml` `version:` semver MUST equal the tag** (tag `v2.0.1` → `version: 2.0.1+…`). The app reports its version from `pubspec` (via `package_info_plus`); the "latest" comes from the tag. If they disagree on a release the updater nags forever (pubspec lower) or never prompts (pubspec higher). Tag the commit *after* the pubspec bump lands.
- **`+<versionCode>`** = `major*10000 + minor*100 + patch` (`2.0.1` → `+20001`) so Android's versionCode always increases with the semver — needed for the updater's install to be a clean "update", not a same-code reinstall.
- Tag names are `vX.Y.Z` only (CI matches `v*`; the updater strips a leading `v`).
- Sequence: bump `pubspec.yaml` → commit → `git tag vX.Y.Z` (on that commit) → `git push origin main && git push origin vX.Y.Z`.
- The `android` CI job signs with the stable release keystore from encrypted secrets (see the signing note under Cloud sync) — keep those set so released APKs carry the OAuth-registered SHA-1.

## Design reference

`references/` holds the design system: `references/gravity_music/DESIGN.md` is the full spec ("Cinematic Dark" / glassmorphism, obsidian-black OLED palette, 30px backdrop blurs, floating "suspended" layout where no element touches screen edges). Each screen folder (`home/`, `library/`, `now_playing/`, `search/`) has a `code.html` Tailwind mockup + `screen.png`. The design tokens live in code as `AppColors` / `AppText` / `AppSpacing` in `lib/ui/app_theme.dart` and the glass primitives in `lib/ui/theme/glass.dart` — use those, not raw values.

## Architecture

### Backend is fully on-device (no remote API)

The app once used a hosted "SaraGama" API (a Render server wrapping `ytmusicapi`). That is **gone** — everything now runs on the device. The pivot point is **`YtMusicService`** (`services/yt_music_service.dart`), a client that calls music.youtube.com's internal `youtubei` endpoints directly (search with the "Songs" filter, `next`/radio for queues, and `browse` for album pages). It returns clean Art Tracks (real titles/artists/album + square `googleusercontent` art). Its response is a deeply-nested `…Renderer` tree parsed defensively, so one shape change doesn't discard the whole result. **`SearchService`, `AlbumService`, `RecommendationService`, `MixesService`, and `ImportService` are all thin layers over `YtMusicService`** — when "up next" or search behaves oddly, suspect the youtubei parsing here.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IamThejus/Gravity-Music](https://github.com/IamThejus/Gravity-Music) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
