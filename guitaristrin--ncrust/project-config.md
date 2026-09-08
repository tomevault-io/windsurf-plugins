---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Build Commands

```bash
./gradlew assembleDebug              # Build debug APK → app/build/outputs/apk/debug/
./gradlew assembleRelease            # Build release APK (signing config is in build.gradle.kts)
./gradlew test                       # Run unit tests
./gradlew connectedAndroidTest       # Run instrumented tests (requires connected device/emulator)
```

- Target/Compile SDK: 36 (Android 15), Min SDK: 24
- Java 11, Kotlin 1.9.24, Compose BOM 2024.12.01

## Development Log & Commit Convention

**No separate log file.** All development history lives in `git log`. Do not create or maintain a parallel `*_log.md` file — a previous `Ncrust_log.md` was removed in favour of git history.

Commit messages follow **Conventional Commits** with a lowercase type prefix:

| Prefix | When to use |
|---|---|
| `feat:` | User-visible new capability (features, UI additions, new APIs) |
| `fix:` | Bug fix; no new behaviour beyond restoring correctness |
| `chore:` | Housekeeping (delete unused files, rename directories, gitignore updates) |
| `docs:` | Docs-only changes (README, AGENTS.md, in-code comments) |
| `build:` | Build system / dependencies / version bumps |
| `refactor:` | Code shape change without behavioural change |
| `perf:` | Performance-only optimisation |
| `style:` | Formatting, whitespace, comment tweaks |

Subject line: prefix + one-sentence Chinese summary. Body (blank line, then paragraphs) explains *why* — Ncrust commits are meant to be readable a year later without opening a PR. Reference issues with `Fixes #N` or `#N` when relevant.

### 提交规范（必须遵守）

- Conventional Commits 前缀：`feat:` / `fix:` / `docs:` / `chore:` / `refactor:` / `test:` 等
- 提交正文使用**中文**
- **一个 section（逻辑单元）一个 commit，主动提交，不等待用户要求**
- 禁止一个 commit 塞多个不相关功能（如「提取 txt + 修颜色 + 改文档」），难看且难回滚
- 一个 commit 只做一件事：改一个工具 / 解一个格式 / 写一份文档 / 更新一类标注

## Versioning

Single source of truth: `app/build.gradle.kts` → `defaultConfig.versionName` (and `versionCode`).

The About page (`ui/screen/AboutScreen.kt`) reads the version dynamically from `BuildConfig.VERSION_NAME` — **never hardcode a version constant here**. This requires `buildFeatures.buildConfig = true` in `app/build.gradle.kts`.

Release flow: bump `versionCode` + `versionName` → commit as `build: 升级至 vX.Y.Z ...` → `./gradlew assembleRelease` → `gh release create vX.Y.Z --draft <apk>` → user manually publishes after smoke test.

## What This App Is

Ncrust is a third-party NetEase Cloud Music (网易云音乐) Android client built around three design priorities:
1. **Kanesumi Design** — right-angle cuts, no curves, no rounded corners, information-first
2. **GPU zero-recomposition** — animations driven by a single `progress: Float` through `graphicsLayer`, not state-driven recomposition
3. **Three-layer graphics architecture** — main page / player card / navigation bar are independent composable layers, enabling gesture transitions without interference

## Terminology: Kanesumi Design

The design language is officially **Kanesumi Design** (canon: Ether monorepo root `KANESUMI_DESIGN.md`), replacing the historical name "Metro Design". Notes for agents:

- `Metro*` identifiers (`MetroText`, `MetroTheme`, `MetroIndication`, easing constants `MetroDefault`/`MetroCubic`…) are code/component names — **keep, do not rename**.
- New docs/comments: write 「Kanesumi Design / Kanesumi 风格」, not 「Metro Design / Metro 风格」. The easing family is 「UWP 缓动」 (`UwpEasing`), from the Metro era.
- Full mapping: `KANESUMI_DESIGN.md` §Ⅴ.

## Architecture

### Entry Point & Structure

`MainActivity.kt` is now a lean entry point (~50 LOC for the activity class itself) that calls `MainScreen()`. All app-level orchestration (navigation, player state, queue management, bottom tab bar) lives in the `MainScreen()` composable in the same file.

Package layout under `com.takahashirinta.ncrust/`:

| Package | Purpose |
|---|---|
| `network/` | Retrofit interface, eapi encryption, response models |
| `player/` | ExoPlayer service, playback state persistence, URL fetching |
| `ui/navigation/` | `NavRoutes` route constants and `MainNavGraph` composable |
| `ui/player/` | Full-screen player card split across `PlayerCardOverlay`, `PlayerCard`, `FullPlayerControls`, `LyricsView`, `QueueView`, `SlimProgressBar` |
| `ui/screen/` | One file per screen (Home, Search, Library, Album/Artist/Playlist detail, etc.) |
| `ui/viewmodel/` | `PlayerViewModel`, `SearchViewModel`, `SongViewModel` |
| `ui/components/` | Reusable composables (`SongCard`, `DetailScaffold`, `PlayAllCircleButton`) |
| `ui/theme/` | Theme color system, `MarkdownText` composable |
| `ui/i18n/` | Runtime i18n system: `Strings` data class, per-language files, `LanguageManager` |
| `ui/anim/sokuou/` | Sokuou animation system: UWP easing family + Apple-style spring presets + `sokuouSpring(response, damping)` bridge |
| `auth/` | Cookie singleton (MUSIC_U extraction, SharedPreferences storage) |
| `library/` | Cloud-synced favorites: 「收藏单曲」= NetEase like API, 「收藏专辑」= 云端专辑收藏; local SharedPreferences cache + background refresh |
| `lyric/` | LRC parser: `[MM:SS.mm]` → `LrcLine.timeMs` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GuitaristRin/Ncrust](https://github.com/GuitaristRin/Ncrust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
