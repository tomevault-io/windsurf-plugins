---
trigger: always_on
description: Guidance for AI coding agents working in the Immortal repository. This is the canonical
---

# AGENTS.md

Guidance for AI coding agents working in the Immortal repository. This is the canonical
agent guide; tool-specific files (e.g. `CLAUDE.md`) just point here. Human contributors
should read [`CONTRIBUTING.md`](CONTRIBUTING.md) — this file is a superset aimed at agents.

## What this project is

Immortal is a custom home-screen layer (launcher + screensaver + app store + fleet tooling)
for discontinued Meta Portal devices. It is a **single Android app**, package
`com.immortal.launcher`, Jetpack Compose, Kotlin, `minSdk 24` / `targetSdk 36`, targeting
Portal hardware on Android 9 (API 28) and Android 10 (API 29), arm64, **no Google services**.
See [`README.md`](README.md) for the full feature tour.

## Build, test, and validate

Always run the relevant check below after a change and fix failures before finishing.

| Task | Command |
|------|---------|
| Build the app (debug) | `./gradlew :app:assembleDebug` |
| Run unit tests (CI gate) | `./gradlew :app:testDebugUnitTest --no-daemon` |
| Install to a connected Portal | `adb install -r app/build/outputs/apk/debug/app-debug.apk` |
| Launch | `adb shell am start -n com.immortal.launcher/.HomeActivity` |
| Validate the store catalog | `python3 scripts/validate_catalog.py --network` |
| Lint provisioning scripts | `bash -n provisioning/provision.sh` |
| Build the fleet CLI | `cd provisioning && rustc -O fleet.rs -o fleetctl` |

CI workflows live in [`.github/workflows/`](.github/workflows/): `tests.yml` (unit tests),
`catalog.yml` (catalog schema + network checks), `provisioning.yml` (bash/PowerShell parse +
ASCII guard), `docs.yml`, `release-guard.yml`.

The debug build uses a `.debug` application-id suffix so it installs alongside a provisioned
release.

## Repository layout

```
app/                         The Android app (single module)
  src/main/java/com/immortal/launcher/   ~99 Kotlin files, flat package, grouped by name prefix
  src/main/assets/           bundled catalog.json fallback, clock faces, fonts, fallback photos
  src/main/res/              Compose theme lives in .../launcher/ui/theme/
  src/test/java/             unit tests
  build.gradle.kts           app build config (signing via keystore.properties)
provisioning/                Provisioning kit + the fleet CLI
  provision.sh / provision.ps1   one-double-click device setup (macOS/Linux, Windows)
  fleet.rs                   source of the fleet CLI (std-lib-only Rust, no crates)
  fleetctl                   prebuilt fleet CLI binary
  fleet-backup.sh / fleet-restore.sh   snapshot/restore a Portal's app data
  fleet/<serial>.json        device registry — SECRETS, git-ignored, never commit
  config.env                 provisioning options
docs/                        MkDocs site; docs/features/*.md and docs/design/*.md
scripts/                     validate_catalog.py, cut-release.sh, check-version-sync.sh, …
catalog.json                 hosted app-store catalog (schema v2)
version.json                 self-update manifest (versionCode/versionName + apkUrl)
skills/                      portable agent skills, Agent Skills SKILL.md format (see below)
```

### Finding code by feature

The Kotlin package is flat; files are grouped by name prefix:

- **Launcher / home grid:** `HomeActivity`, `UserLayout`, `QuickBar*`, `AppSwitcherActivity`
- **Screensaver / photo frame:** `PhotoDreamService`, `PhotoFrameController`, `Screensaver*`,
  `Face*` / `ClockFaces` / `FlipWebClockFaceView`, `DreamPolicy`, `PresenceState`, `AntiBurnIn`;
  digital-clock face `DigitalClock*`; welcome overlay `Welcome*`; Dream selection in `SettingsGuard`
- **Photo sources:** `LocalMedia`, `ImmichSource`, `SmbSource`, `DavSource`, `RemoteAlbum`,
  `Weather`, `CalendarFeed`
- **Tools screen:** `ToolsActivity`, `HomeToolOverlays`, `CameraViewerActivity` / `CameraConfig`,
  `LampActivity`, `BedtimeStoryActivity` / `Stories`, `IntercomActivity` / `LanAudio`,
  `CountdownSettingsActivity` / `CountdownConfig`, plus the keyless helpers `Converter`, `IssPasses`,
  `Aurora`, `PrayerTimes`
- **Ambient & sound:** chimes `Chime*` / `ChimeConfig`; sunrise wake-light `Sunrise*` /
  `SunriseConfig`; almanac packs `CalendarPacks` (+ `FeastDays`, `NameDays`, `IrishHolidays`,
  `DailyContent`)
- **Wallpaper (home background):** `WallpaperConfig`, `AmbientBackground` / `HomeBackground`,
  `SkyColors`, `StarField`
- **Accessibility / input:** back gesture `BackHelper`, `ImmortalBackGestureService`,
  `SystemBackGestureService`; touch sounds `SystemSounds`
- **App store / install:** `StoreActivity`, `StoreCatalog`, `UpdateManager`, `InstallDaemon`,
  `HeadlessInstaller`, `ApkInstallActivity`, `ApkBrowserActivity`, `InstallConfirmService`
- **Fleet agent (on-device HTTP API):** `FleetAgentService`, `FleetHttpServer`, `FleetRoutes`,
  `FleetConfig`, `FleetFs`, `FleetDiag`, `FleetCalendar`, `FleetScreensaver`
- **Multi-room audio / now playing:** `MultiRoom*`, `NowPlaying*`, `Snapcast*`, `Ma*`,
  `MediaSession*`, `MediaNotificationListenerService`
- **Smart home (MQTT):** `Mqtt*`; ambient sensor entities `AmbientSensors`; presence read from
  Meta's own detector `PortalPresence` (feeds `PresenceHub`)
- **Remote / Portal TV:** `Remote*`, `TvFocus`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [starbrightlab/immortal](https://github.com/starbrightlab/immortal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
