---
trigger: always_on
description: Vimm's Lair download queue manager with PS3 ISO conversion and sync. Modular architecture under `Modules/`. React + Vite + Tailwind frontend in `VimmsDownloader/client/`. Host project `VimmsDownloader/`. Mock server in `MockServer/` for testing. GitHub user: eduvhc. Target platform: Linux (Docker + bare metal), also runs on Windows for dev.
---

# CLAUDE.md

## Project Overview

Vimm's Lair download queue manager with PS3 ISO conversion and sync. Modular architecture under `Modules/`. React + Vite + Tailwind frontend in `VimmsDownloader/client/`. Host project `VimmsDownloader/`. Mock server in `MockServer/` for testing. GitHub user: eduvhc. Target platform: Linux (Docker + bare metal), also runs on Windows for dev.

## Architecture

### Modules (under `Modules/`)

All modules follow the convention in `Modules/MODULE_GUIDE.md`. Each module is a standalone class library with zero web dependencies, communicating with the host via a typed bridge (`IModuleBridge<TEvent>`).

- **Module.Core** — `IModuleBridge<TEvent>` interface, `Result<T>` (generic result type + `FileOps` safe I/O helpers), `SharedConstants` (`FileExtensions`, `Platforms`), and `Pipeline/` infrastructure (`IPipeline`, `PipelineState`, `PipelinePhase`, `PipelineStatusEvent`, `PipelineFlowInfo`, `DuplicateCheckResult`). Every module references this.
- **Module.Core.Testing** — Shared test infrastructure: `FakeBridge<T>`, `TempDirectory`, `ToolsContainer` (Testcontainers with `ghcr.io/eduvhc/vimm-dl-tools`).
- **Module.Download** — Download service: `DownloadService` (download loop, resume, progress, Result-based error handling), `VaultPageParser` (HTML parsing + format resolution with fallback), `IDownloadItemProvider` (async, host provides queue items). Bridge: `IDownloadBridge` emitting `DownloadStatusEvent`, `DownloadProgressEvent`, `DownloadCompletedEvent`, `DownloadErrorEvent`, `DownloadDoneEvent`.
- **Module.Extractor** — 7z wrapper (`ZipExtract.QuickCheckAsync`, `ExtractAsync` returning `Result<bool>`). Auto-detects `7z` on PATH or `C:\Program Files\7-Zip\7z.exe` on Windows.
- **Module.Ps3IsoTools** — Pure PS3 tools, no pipeline: `ParamSfo` (PARAM.SFO binary parser), `Ps3IsoConverter` (makeps3iso + patchps3iso + `FindJbFolder`, returns `Result<string>`), `IsoFilenameFormatter` (serial/The-fix/region rename with `IsoRenameOptions`).
- **Module.Ps3Pipeline** — PS3 pipeline orchestration, implements `IPipeline`: `Ps3ConversionPipeline` (facade with `BuildFlow`, `CheckDuplicate`, `GetStepDurations`), `Ps3JbFolderPipeline` (extract→convert workers), `Ps3DecIsoPipeline` (rename/extract .dec.iso, optional archive preservation). Uses `PipelineState` from Module.Core. Bridge: `IPs3PipelineBridge : IModuleBridge<PipelineStatusEvent>`.
- **Module.Sync** — Compares ISOs between `completed/` and an external drive. Copy with progress, disk info, space checks via `ISyncBridge`.

### Host (VimmsDownloader/)

- **SRP file structure** — `Program.cs` (startup/DI), `Models.cs` (records + PathHelpers), `AppJsonContext.cs` (JSON source gen), `QueueRepository.cs`, `SettingsKeys.cs`, `DatabaseMigrator.cs` (embedded SQL migrations), `DownloadHub.cs`, `DownloadQueue.cs`, `QueueItemProvider.cs`, `MetadataFetcher.cs`.
- **Endpoints/** — `FileEndpoints` (merged `/api/data` with pipeline trace), `DownloadEndpoints`, `MetadataEndpoints`, `Ps3Endpoints`, `SyncEndpoints`, `SettingsEndpoints`, `EventEndpoints`, `MetricsEndpoints`. 21 endpoints total.
- **SignalR bridges** — `SignalRPs3PipelineBridge.cs`, `SignalRSyncBridge.cs`, `SignalRDownloadBridge.cs` route module events to SignalR + append to events table. Pipeline bridge also updates `completed_urls` projection for terminal states.
- **AOT-ready** — `PublishAot=true`, raw ADO.NET, JSON source generator, all modules `IsAotCompatible`.
- **QueueRepository** — singleton, all async SQLite operations. Database initialized via `DatabaseMigrator` with embedded SQL files.
- **Settings stored in DB** — `settings` table (key-value). Keys in `SettingsKeys.cs`.

### Result Pattern

- `Result<T>` struct in `Module.Core/Result.cs` — zero-allocation, `IsOk`/`Error`/`Value`
- `FileOps` utility — `TryMove`, `TryDelete`, `TryDeleteDirectory`, `TryWriteAllText`
- Modules return `Result<T>` instead of throwing for expected failures
- Exceptions reserved for critical failures + `OperationCanceledException` (cancellation mechanism)

### Event Sourcing

- `events` table — append-only log of ALL module events (download, pipeline, sync) including progress
- `correlation_id` column links all events for a single pipeline run (12-char hex UUID)
- Bridges write to events table before SignalR dispatch
- `completed_urls` is a projection — `conv_phase`/`conv_message`/`iso_filename` updated by pipeline bridge on terminal events
- Events pruned on startup: 7-day retention + 50k max rows

### Database Migrator

- `DatabaseMigrator.cs` — runs embedded SQL files from `Migrations/*.sql` in order
- Tracks executed migrations in `schema_migrations` table
- Each migration is idempotent (catches "duplicate column" / "already exists" errors)
- Migrations split into individual statements for SQLite compatibility

## Database

SQLite, file `queue.db` in `data/` subdirectory (derived from connection string). Seven tables:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eduvhc/vimm-dl](https://github.com/eduvhc/vimm-dl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
