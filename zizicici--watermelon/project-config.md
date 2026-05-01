---
trigger: always_on
description: This file is the canonical project guide for coding agents working in this repository. Claude Code reads it through the `CLAUDE.md` symlink; OpenAI Codex and other agents that honor the `AGENTS.md` convention read it directly.
---

# AGENTS.md

This file is the canonical project guide for coding agents working in this repository. Claude Code reads it through the `CLAUDE.md` symlink; OpenAI Codex and other agents that honor the `AGENTS.md` convention read it directly.

## Project Overview

Watermelon is an iOS photo backup app that reads from `PHAsset` and writes to remote storage (`SMB`, `WebDAV`, or an external-volume folder via security-scoped bookmark).

The app currently centers on a single Home screen plus a More/settings page. Home is no longer a fat view controller; the runtime flow is split across `HomeViewController`, `HomeScreenStore`, `HomeConnectionController`, `HomeExecutionCoordinator`, and `HomeIncrementalDataManager`.

Build with `Watermelon.xcodeproj`. There is no automated test suite in the repo; critical-path validation is still manual.

## Source Layout

```text
Watermelon/
  App/          # AppDelegate, SceneDelegate, AppCoordinator, AppSession,
                # DependencyContainer, ProStatus, RatingPromptService
  Home/         # HomeViewController, HomeScreenStore, HomeScreenState,
                # HomeDataModels, HomeCollectionViewCells,
                # HomeConnectionController, HomeExecutionCoordinator,
                # HomeExecutionSession, HomeExecutionDataRefresher,
                # HomeExecutionLogViewController, ExecutionLogEntryCell,
                # ExecutionLogHistoryViewController,
                # HomeLibraryEngines, HomeAlbumMatching,
                # DownloadWorkflowHelper,
                # SelectionActionPanel, SelectionActionPanelRenderState
  Services/
    Backup/     # BackupCoordinator, BackupRunPreparation, BackupParallelExecutor,
                # BackupSessionController, BackupSessionAsyncBridge,
                # BackupSessionReducer, BackupRunDriver, BackupResumePlanner,
                # BackupMonthScheduler, BackupCancellationController,
                # BackupAssetResourcePlanner, BackgroundBackupRunner,
                # BackupEvent, BackupEventStream,
                # BackupRunModels, BackupScopeModels,
                # StorageClientPool,
                # AssetProcessor (+Upload, +Naming), AssetProcessModels,
                # MonthManifestStore (+Loading, +Schema),
                # RemoteIndexSyncService, RemoteLibrarySnapshotCache
    HashIndex/  # ContentHashIndexRepository, LocalHashIndexBuildService
    Logging/    # ExecutionLogEntry, ExecutionLogFileStore,
                # ExecutionLogSessionInfo, ExecutionLogSessionWriter
    PiP/        # PiPExecutionBridge, PiPLogTailRenderer, PiPProgressManager
    PhotoLibrary/
    Restore/
    SMB/
    Storage/
  UI/
    Auth/       # storage profile create/edit flows
    More/       # WatermelonMoreDataSource, Settings
    Common/
  Data/
    Database/   # DatabaseManager, Records
    Security/   # KeychainService
  Domain/       # backup/storage/remote snapshot domain models
  Extension/    # AppColor, ConsideringUser, hex/time/localization helpers
  Resource/     # Localizable.xcstrings, assets
```

## Key Files (read in this order)

1. `Watermelon/App/DependencyContainer.swift`
2. `Watermelon/Home/HomeViewController.swift`
3. `Watermelon/Home/HomeScreenStore.swift`
4. `Watermelon/Home/HomeConnectionController.swift`
5. `Watermelon/Home/HomeExecutionCoordinator.swift`
6. `Watermelon/Home/HomeExecutionSession.swift`
7. `Watermelon/Home/HomeLibraryEngines.swift`
8. `Watermelon/Services/HashIndex/LocalHashIndexBuildService.swift`
9. `Watermelon/Services/Backup/BackupSessionController.swift`
10. `Watermelon/Services/Backup/BackupRunPreparation.swift`
11. `Watermelon/Services/Backup/BackupParallelExecutor.swift`
12. `Watermelon/Services/Backup/AssetProcessor.swift`
13. `Watermelon/Services/Backup/RemoteIndexSyncService.swift`
14. `Watermelon/Services/Restore/RestoreService.swift`

## Architecture

### App Startup

`SceneDelegate` -> `AppCoordinator.start()` -> `HomeViewController`.

There is no global TabBar and no root `UINavigationController`; `HomeViewController` is set directly as the window root.

### Dependency Injection

`DependencyContainer` owns the top-level services:

- `DatabaseManager`
- `KeychainService`
- `AppSession`
- `StorageClientFactory`
- `PhotoLibraryService`
- `ContentHashIndexRepository`
- `LocalHashIndexBuildService`
- `BackupCoordinator`
- `RestoreService`

`AppSession` stores the active profile and in-memory session password. SMB/WebDAV require passwords; external volume does not.

### Home Layering

#### `HomeViewController`

UI-only layer for:

- two-column month grid
- top headers and profile menu
- right-side remote overlay (`connecting` / `disconnected`)
- bottom `SelectionActionPanel`
- floating More/settings button

It binds to `HomeScreenStore.onChange` and renders seven change kinds:

- `.data`
- `.fileSizes`
- `.selection`
- `.execution`
- `.connection`
- `.connectionProgress`
- `.structural`

#### `HomeScreenStore`

State aggregator for Home. Owns:

- `HomeIncrementalDataManager`
- `HomeConnectionController`
- `HomeExecutionCoordinator`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zizicici/Watermelon](https://github.com/zizicici/Watermelon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-27 -->
