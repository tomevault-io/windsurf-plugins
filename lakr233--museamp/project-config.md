---
trigger: always_on
description: - This is a pure UIKit iOS app with programmatic setup. `MuseAmpTV/` is the tvOS shell target.
---

# Project Overview

## App Shape

- This is a pure UIKit iOS app with programmatic setup. `MuseAmpTV/` is the tvOS shell target.
- Do not introduce SwiftUI for new UI work; all new screens, components, and overlays must use UIKit.
- App entry remains `MuseAmp/main.swift` -> `Application/AppDelegate.swift` -> `Application/SceneDelegate.swift`.
- TV app entry should remain `MuseAmpTV/main.swift` -> `MuseAmpTV/Application/TVAppDelegate.swift` -> `MuseAmpTV/Application/TVSceneDelegate.swift`.
- `Interface/Root/TabBarController.swift` owns the shared `AppEnvironment` reference for the compact tab shell.
- Compact root navigation currently uses first-level UIKit tabs for `Albums`, `Songs`, `Playlist`, optional `Search`, and `Settings`.
- Relaxed/Catalyst root navigation currently uses sidebar destinations for `Albums`, `Songs`, `Downloads`, `Playlist`, optional `Search`, and `Settings`.
- `Now Playing` is presented through `LNPopupController`, not as a first-level tab.

## Structure Rules

### Top Level

- `MuseAmp/` contains the app target source and resources.
- `MuseAmpTV/` contains the tvOS shell source and resources. Keep it limited to lifecycle/bootstrap plus the root state-machine shell and thin backend adapters.
- `MuseAmpDatabaseKit/` is the local library runtime package for file-backed library indexing, state, caches, downloads, playlists, and audit data.
- `SubsonicClientKit/` is a separate package for remote music service integration.
- `MuseAmpInterfaceKit/` is a separate UIKit package for cross-platform interface primitives and feature-facing UI contracts shared by iOS/iPadOS/tvOS.
- `MuseAmpPlayerKit/` is a separate package for playback engine integration.

### Application Layer

- `Application/` contains only app lifecycle and configuration: `AppDelegate.swift`, `SceneDelegate.swift`, `AppEnvironment.swift`, `AppEnvironment+Bootstrap.swift`, `AppEnvironment+Events.swift`, `AppEnvironment+Transfer.swift`, `AppPreferences.swift`.
- `MuseAmpTV/Application/` contains only tvOS lifecycle/bootstrap and the thin TV app context. Do not move iOS shell lifecycle types here, and do not grow it into a second full app environment layer.
- `Configuration/` (project root) contains shared Xcode build configuration files (`Base.xcconfig`, `Development.xcconfig`, `Release.xcconfig`, `Version.xcconfig`) used by both MuseAmp and MuseAmpTV targets.
- All domain logic lives under `Backend/`, not `Application/`.

### Backend Layer

- `Backend/API/`: `APIClient` and intent-level network entry points.
- `Backend/Downloads/`: download orchestration plus persisted download records.
- `Backend/Library/`: local and remote library data providers that bridge app services into browseable media collections.
- `Backend/Logging/`: file-backed logging and log reading. Extend this instead of adding a second diagnostics path.
- `Backend/Lyrics/`: lyrics fetching, parsing, and Chinese script conversion.
- `Backend/MenuProviders/`: shared UIKit menu and action provider helpers used by feature controllers.
- `Backend/Models/`: app-facing media models and adapter types used by the interface layer, including shared export/transfer models such as `SongExportItem`.
- `Backend/Playback/`: playback state ownership, playback domain models, and the shared `PlaybackController` bridge to `MuseAmpPlayerKit`.
- `Backend/Playlist/`: playlist CRUD, persistence, artwork generation, and `PlaylistCoverArtworkCache`.
- `Backend/Sync/`: local-network song transfer protocol models, sender/receiver session orchestration, Bonjour discovery/advertising, and transfer file preparation.
- `Backend/Supplement/`: cross-cutting utilities (metadata helpers, string sanitization, file name formatting, concurrency helpers).

### Interface Layer

- `Interface/Root/`: app shell hosting and root composition.
- `MuseAmpTV/Interface/Root/`: tvOS root state machine, shared-TV-screen hosting, and thin backend adapter wiring. Keep focus/navigation logic here rather than in `MuseAmpInterfaceKit`.
- `Interface/Root/BootProgressController.swift`: blocking library boot controller that initializes the local runtime before presenting the tab shell.
- `Interface/Common/`: shared loading, error, and infrastructure UI.
- `Interface/Common/` may contain thin compatibility shims that forward old app-target type names into `MuseAmpInterfaceKit` while feature migration is in progress.
- `Interface/Collections/`: reusable album and track presentation components.
- `Interface/Common/Presenters/SongExportPresenter.swift`: UI presenter for sharing/exporting songs (split from `Backend/MenuProviders/`).
- `Interface/Browse/`: browse-domain UIKit flows for local library navigation and drill-in screens.
- `Interface/Browse/Albums/`: album-library entry screens and their responsibility-based extensions.
- `Interface/Browse/Songs/`: song-library entry screens and their responsibility-based extensions.
- `Interface/Browse/AlbumDetail/`: album detail screens, album-only cells, and related presentation helpers.
- `Interface/Browse/Downloads/`: downloads browse UI and download-list presentation.
- `Interface/Browse/Support/`: browse-only helpers that are shared across album/song/download flows.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lakr233/MuseAmp](https://github.com/Lakr233/MuseAmp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
