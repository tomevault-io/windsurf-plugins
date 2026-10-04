---
trigger: always_on
description: An Android app that downloads/updates the Retro Rewind Mario Kart Wii Pack and launches Dolphin Emulator.
---

# WheelWitch

An Android app that downloads/updates the Retro Rewind Mario Kart Wii Pack and launches Dolphin Emulator.

> Build instructions, commit conventions, and contribution guidelines live in [CONTRIBUTING.md](CONTRIBUTING.md). This file covers architecture, key decisions, constants, and testing: the things that don't change often.

## Build & Dev

See [CONTRIBUTING.md#build](CONTRIBUTING.md#build) for build commands and signing setup.

### Formatting (Spotless + ktfmt)

- `./gradlew spotlessApply`: auto-format all `.kt` and `.kts` files
- `./gradlew spotlessCheck`: verify formatting (for CI)
- No configuration to debate; ktfmt DEFAULT style is enforced

### Linting (Android Lint)

- `./gradlew lint`: runs Android Lint on the default variant
- `./gradlew check`: includes lint + unit tests
- Config in `app/lint.xml` (silences non-actionable checks)
- Keep lint clean before committing; no errors allowed

## Git & Commits

- Commit messages follow [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/): `<type>(<scope>): <description>`, lowercase imperative, no trailing period. See [CONTRIBUTING.md#commit-messages](CONTRIBUTING.md#commit-messages) for types and scopes.
- Stage only intended files with `git add <file>`; read `git status` before every commit; never use `git add -A` or `git commit -a` without verifying the diff.
- Do not add wordy commit messages, a short description is more than enough.

## Architecture

- **Min SDK**: 31, **Target SDK**: 36, **Java 11**, **Compose + Material3** with dynamic color
- **No Google Play Services**: sideloaded APK only
- **Landscape-locked** fullscreen via `WindowCompat.getInsetsController`
- **Navigation**: flat overlay pattern via `AnimatedVisibility`. No NavHost.
- **Gamepad focus isolation**: background screens removed from composition when overlays active
- **OkHttp 4.12.0** via `HttpClientProvider` singleton (15s / 60s read timeout variants)
- **SAF folder picker** for storage; `DolphinTree` wraps the SAF grant + `DocumentFile` operations
- **Dolphin launch**: `AutoStartFile` intent extra + `Dolphin.ini` `ISOPathN` library registration; `DolphinLauncher.launchRetroRewind()` orchestrates the full pre-launch flow with fallback
- **i18n**: all strings in `res/values/strings.xml`; Compose uses `stringResource()`, VMs use `app.getString()`
- **Icons**: Material Symbols (Rounded) vector drawables in `res/drawable/ic_*.xml`, sourced from [fonts.google.com/icons](https://fonts.google.com/icons) (Android tab, 24dp). Loaded in Compose via `ImageVector.vectorResource(R.drawable.ic_x)`. Auto-mirrored icons (`arrow_back`, `exit_to_app`, `shortcut`) declare `android:autoMirrored="true"` on the root `<vector>` to mirror in RTL locales. The deprecated `androidx.compose.material.icons` artifacts are intentionally not used.

## Package Structure

```
com.skiletro.wheelwitch
├── MainActivity.kt
├── model/         (data types: SemVersion, PackStatus, UpdateEntry, DeletionEntry, SaveFileInfo, LicenseInfo, ServerInfo, etc.)
├── data/          (storage: DolphinPaths, DolphinTree, DolphinConfig, SaveManager, RksysParser, GameTypeParser)
├── network/       (HTTP + JSON parsers: VersionFileParser, RoomStatusParser, RaceStatsParser, ServerHealthParser, LeaderboardParser, TimeTrialParser)
├── domain/        (business logic: RewindPackManager)
├── util/{io,net,mii,launcher,log,json,prefs}/   (utilities grouped by concern)
├── ui/{components,screens,theme}/
└── viewmodel/
    ├── PackUpdateViewModel   : install/update state machine + SAF tree wiring
    ├── SaveDataViewModel     : per-region parse, leaderboard merge, backup/restore/delete, multi-region
    ├── MiiMakerViewModel     : WAD install (Mutex-guarded), launch, delete
    ├── OnlineViewModel       : rooms, leaderboard (Channel-based race-free), health, race stats
    └── UiState               : sealed class for pack update flow
```

The sub-packages under `util/` are intentional. Keep new files in the right sub-package:
- `io/`: `FileDownloader`, `ByteReader`, `OptionalFileTree`
- `net/`: `HttpClientProvider`, `NetworkExtensions`
- `mii/`: `MiiWadInstaller`, `MiiFaceCache`, `MiiEndpoints`
- `launcher/`: `DolphinLauncher`, `BugReportLauncher`
- `log/`: `LogBuffer`, `LogEntry`, `LogExporter`, `MemoryBufferTree`, `AppReleaseLogTree`
- `json/`: `JsonExtensions`
- `prefs/`: `Prefs`, `PrefsKeys`

## Key Decisions

- **Screen nav**: `MainScreen` orchestrates `HomeScreen`/`SettingsScreen`/`OnboardingScreen`; `HomeScreen` orchestrates `OnlineMenuScreen`/`SaveInfoScreen`; `OnlineMenuScreen` uses `AnimatedContent` with `OnlineMenuPage` enum
- **Dolphin tree**: `DolphinTree` (SAF wrapper) is the single source of truth for the user-picked folder; `DolphinPaths` derives physical paths via the package-swap trick; `DolphinConfig` is the pure INI parser for `Dolphin.ini` `ISOPathN` registration
- **Path consistency invariant**: every path inside `rr_autostartfile.json` and the `AutoStartFile` extra must derive from the same `DolphinPaths.physicalRoot(context)` call. Riivolution's native code can't resolve `content://` URIs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TeamWheelWizard/WheelWitch](https://github.com/TeamWheelWizard/WheelWitch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
