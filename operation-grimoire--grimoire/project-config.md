---
trigger: always_on
description: This file is the orientation a Claude agent should read before editing the Grimoire app. It complements [`README.md`](./README.md) (public-facing intro) with the gotchas, conventions, and "where do I find X" that aren't obvious from the source tree.
---

# CLAUDE.md — Agent guide for the Grimoire app

This file is the orientation a Claude agent should read before editing the Grimoire app. It complements [`README.md`](./README.md) (public-facing intro) with the gotchas, conventions, and "where do I find X" that aren't obvious from the source tree.

**Also read [`CONTRIBUTING.md`](./CONTRIBUTING.md)** — it covers environment setup, the build/test commands, and the contribution conventions (branching, PRs, the "don'ts") for both humans and agents. This file layers the deeper architecture map and per-screen gotchas on top; it does not repeat the basics there.

If something here drifts from the code, **trust the code and update this file**.

## Tech baseline

- **Kotlin 2.2.10** / KSP 2.2.10-2.0.2 / AGP 9.2.1
- **Compose BOM 2026.02.01**, Material 3
- **minSdk 26**, **targetSdk 36** (compileSdk API 36.1), Java 11 / jvmTarget 11
- Hilt 2.59.2, Room 2.7.1, Navigation-Compose 2.8.9, Coil 2.7, OkHttp 4.12, DataStore 1.1, WorkManager 2.10
- **Icons are Material Symbols (Outlined)**, not `material-icons-extended` (that dependency is removed). Every icon is an `AppIcons.<Name>` extension `val` in `ui/icon/`, fetched from Google's Material Symbols render endpoint. Reference `AppIcons.Search` etc.; never `Icons.Default.*`.
- `androidx.media` 1.7 is the legacy `MediaSessionCompat` — TTS still uses it; do **not** introduce media3 in a single screen without migrating the whole TTS playback service at once
- `biometric:1.2.0-alpha05` + `security-crypto:1.1.0-beta01` are intentionally alpha/beta; watch for breakage on bumps
- Release builds run **without R8** (`isMinifyEnabled = false`). Don't enable minify in a one-off PR without adding the Hilt / Room / serialization keep rules

## Package layout — io.grimoire.app

```
ui/                  Compose UI (Scaffold, NavHost, screens, ViewModels)
  AppNavigation.kt     Top-level Scaffold + bottom nav + NavHost
  AppNavGraphs.kt      Per-feature NavGraphBuilder extensions
  AppRoutes.kt         Route constants + TopLevelDestination
  component/           Shared composables
  screen/              browse / downloads / extensions / library / migrate /
                       more / novelupdates / reader / settings (15 subscreens) /
                       tasks / updates / webview
  theme/, update/
data/
  local/               Room: AppDatabase, 6 DAOs, 7 entities
  preferences/         14 DataStore-backed pref classes + PreferenceStore
  source/              ChapterListFetcher + paginated-source helpers
  download/            DownloadManager / Service / ChapterImageStore
  libraryupdate/       Scheduler / Worker / Updater
  backup/              BackupManager / Scheduler / Worker / Models
  tts/                 13 files — DeviceTts + ElevenLabs + PlaybackManager / Service / Notification
  novelupdates/        NU.com scraper (Client / Parser / Matcher / Endpoints / Models)
  update/              In-app GitHub release updater + Changelog parser
  epub/                EpubImporter / Parser, Local source
  cache/               CoverPreloader
domain/                auth / migration / novelupdates info repositories
extension/             ExtensionLoader / Manager + repo browsing
auth/github/           OAuth device flow (5 files)
di/                    Hilt modules: Database, Preferences, GitHubAuth
util/                  ContentLanguages, LanguageLabels
```

## Architecture conventions

- **MVI-lite per screen**: `XScreen` + `XViewModel` per feature. ViewModels expose `StateFlow`, screens read via `collectAsState`.
- **Filter / sort / search projections belong in the ViewModel.** Use a pure top-level function (`computeTabNovels` in `screen/library/LibraryFilter.kt`, `projectChapters` in `screen/browse/ChapterProjection.kt`) and back it with a derived `StateFlow` via `combine(...).flowOn(Dispatchers.Default).stateIn(...)`. Don't recompute the projection in the screen body on every recomposition.
- **Search input is debounced 120 ms at the VM** (`MutableStateFlow<String>.debounce(120L)`) so each keystroke doesn't refire the projection.
- **Pager pages read precomputed tabs** by index. Never run the per-page filter inside `HorizontalPager`'s page lambda — that triples the work on adjacent-page preload.
- **Tabs are always swipeable.** Any tabbed surface (screens *and* bottom sheets) uses `ui/component/SwipeTabRow.kt` — a `TabRow` + `HorizontalPager` so tabs change on swipe as well as tap. Don't hand-roll a `TabRow` whose content switches via `when`/`if`; reach for `SwipeTabRow` with the matching `SwipeTabStyle` (`Primary` / `PrimaryScrollable` / `Secondary`) and pass `fillHeight = false` inside a sheet. Pass your own `PagerState` when the selection must persist/restore (the library remembers its category).
- **Hidden categories / locked chapters**: the *locked* visibility state hides hidden-category novels from the All tab regardless of `includeHiddenInAll`; the *unlocked* state honours the pref. Tests in `LibraryFilterTest` cover both.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Operation-Grimoire/grimoire](https://github.com/Operation-Grimoire/grimoire) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
