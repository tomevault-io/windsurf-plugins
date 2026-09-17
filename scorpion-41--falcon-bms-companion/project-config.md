---
trigger: always_on
description: Three deliverables live in this repo, all built from the same Kotlin/Compose screens in `app/src/main/java`:
---

# Falcon BMS Companion: notes for AI-assisted maintenance

Three deliverables live in this repo, all built from the same Kotlin/Compose screens in `app/src/main/java`:
1. **Android app** `app/`: Kotlin + Jetpack Compose + Material 3, package `com.bmscompanion.app`, a single APK (charts and every map style/zoom level included).
2. **BMS Companion for Windows** `desktop/`: Compose for Desktop. It reads Falcon BMS (the former C# bridge, ported to Kotlin in `desktop/.../bridge/`), serves the API, the browser version and the bundled data on one port (47474), and is the full app. One window: the light **server page** or the **full app**. Packaged as MSI + portable zip with its own Java runtime.
3. **Browser version** `web/`: Kotlin/Wasm + Compose for Web. The same app running in the browser (iPhone, iPad, any browser); served by the PC program from its resources (`/webapp`).

## Ground rules
- **The BMS install is read-only.** Never write, move or delete anything in the BMS folder, including during testing. To test EZBoards, copy its folder to a temp dir and strip the `SET KNEEBOARD[...]` lines from the copy's `CONFIG_USER.BAT`.
- **No personal data in the repo** (Windows user names, callsigns, absolute user paths, LAN addresses in screenshots). `local.properties`, `dist/`, `posts/` and `tools/pdftext/` are git-ignored. The demo briefing in `desktop/src/main/resources/bridge` is sanitized. Screenshots come from demo mode (env `BMSC_DEMO=1`).
- **Phone layouts must stay usable.** Tablet-specific layouts use `isWide()` (≥840dp) / `isMedium()` (≥600dp), `AdaptiveSplit` and `Masonry` from `ui/components/Adaptive.kt`. The PC window and the browser page feed their width into the same checks.
- **Shared app code must also compile for the PC and the browser.** After changing `app/src/main/java`, run `./gradlew :desktop:compileKotlin :web:compileKotlinWasmJs`. Don't add Android-only APIs to shared screens (stand-ins: `desktop/src/main/kotlin/shims/`, `web/src/wasmJsMain/kotlin/shims/`). Prefer `kotlin.*` over JVM APIs; the browser version only has stand-ins for `String.format` (patterns in `web/.../Printf.kt`), `Math`, `System`, `Locale` casing, `SimpleDateFormat`/`Date`, `synchronized` (per package in `web/.../shims/jvmapis/`).
- **JSON contract changes are additive.** The server serializes the app's `app/.../data/mission/MissionModels.kt` classes directly; update the producing code in `desktop/.../bridge/` and `docs/PROTOCOL.md`.

## Where things are
| Topic | Files |
|---|---|
| Navigation, tabs, routes (`m/...` = pages opened from Mission, owned by the Mission tab); `AppRoot(startRoute, nav)` | `app/.../ui/AppRoot.kt` |
| Bundled data loading, prefs | `app/.../data/Repo.kt`, `data/Models.kt` (PC: `desktop/.../overrides/data/Repo.kt`; browser: `web/.../overrides/data/Repo.kt`, data over HTTP, prefs in localStorage) |
| Theater map widget (projection x=north ft, y=east ft) | `app/.../ui/components/TheaterMap.kt` (PC and browser: `desktop/.../overrides/ui/components/TheaterMap.kt`) |
| Map styles, tile levels, landmarks (borders, provinces, labels, towns), the **Map** menu (`MapLook` prefs), mission towns (`MapFocus`/`MapMission`, set by `PublishMapMission` in `MissionTabs.kt`; `GeoPaths.relevant`) | `app/.../ui/components/MapBase.kt`; data `assets/maps/<mapId>/<style>.webp` + `<style>/<z>/<r>_<c>.webp`, `assets/data/geo/<mapId>.json`; made by `tools/extractor/src/maps.mjs`, `geo.mjs`, `projection.mjs` |
| Tankers & support (TACAN, UHF, location) | `app/.../ui/screens/mission/MissionSupport.kt` (Dashboard card `SUPPORT`, Briefing, Comms) |
| Mission client (polling while screen shown, UDP discovery) | `app/.../data/mission/MissionLink.kt`; PC `desktop/.../overrides/data/mission/MissionLink.kt` (`LinkMode.LOCAL` calls `Bridge.handle` in-process); browser `web/.../overrides/data/mission/MissionLink.kt` (same-origin `/api`) |
| Mission UI | `app/.../ui/screens/mission/*.kt`: tabs and shared plumbing in `MissionTabs.kt` (Dashboard / Map / AWACS / Flight / Briefing / Comms / Boards / Setup); `MissionDashboard.kt` (cards, span masonry, saved layouts), `MissionAwacs.kt` + `AwacsTools.kt` (GCI geometry and calls) |
| Media (BMS screenshots) | `app/.../ui/screens/Media.kt` (grid, viewer), platform share/download actions via `data/ImageActions.kt` (`Platform.imageActions`, `LocalImageActions`; PC `desktop/.../PcImageActions.kt`, browser `web/.../Main.kt`); server side `desktop/.../bridge/Screenshots.kt` (thumbnails, Recycle-Bin delete) |
| Reading BMS: shared memory struct offsets and reader | `desktop/.../bridge/SharedMemory.kt` (mirrors `Tools/SharedMem/FlightData.h`; JNA) |
| Briefing / DTC parsers | `desktop/.../bridge/BmsFiles.kt` |
| AWACS picture | `desktop/.../bridge/TacviewClient.kt` (Tacview real-time telemetry, port 42674) |
| EZBoards | `desktop/.../bridge/EzBoards.kt` (hidden `cmd /c EZBOARDS.BAT companion`; board tables via `bin\xbrief.exe`) |
| API routes, file watching, demo mode, settings (`bridge-settings.json`) | `desktop/.../bridge/Bridge.kt`, `DemoSource.kt`, `BridgeSettings.kt` |
| HTTP server (API, forwarding for clients, browser version, `/assets`), discovery | `desktop/.../PcServer.kt`, `desktop/.../bridge/Discovery.kt` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Scorpion-41/Falcon-BMS-Companion](https://github.com/Scorpion-41/Falcon-BMS-Companion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
