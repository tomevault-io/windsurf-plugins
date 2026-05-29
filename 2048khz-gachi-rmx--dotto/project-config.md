---
trigger: always_on
description: A .NET 9 Discord bot that downloads media from URLs and posts it to Discord. Uses NetCord, PostgreSQL, optional S3 overflow storage, yt-dlp/Cobalt download backends, and ffmpeg video compression.
---

# Dotto - Discord Bot

A .NET 9 Discord bot that downloads media from URLs and posts it to Discord. Uses NetCord, PostgreSQL, optional S3 overflow storage, yt-dlp/Cobalt download backends, and ffmpeg video compression.

## Instructions

Some folders or projects may contain FEATURE.md that describes a feature with additional detail - explanations, implementation quirks, requirements, etc.
You should generally only read them if you're planning to actively work on that feature.
Conversely, if you work on the feature and edit its' mechanics, you should prompt the user to write down the changes in those files when done.

## Architecture

Layered architecture with projects in `Dotto.sln`:

```
Dotto.Bot (entry point)
  └── Dotto.Discord (Discord integration)
        └── Dotto.Application (business logic)
              ├── Dotto.Common (shared utilities)
              ├── Dotto.Downloader.Contracts (downloader interfaces)
              └── Dotto.Ffmpeg.Contracts (compression interfaces)
  └── Dotto.Database (PostgreSQL data access)
  └── Dotto.Downloader (media download implementations)
  └── Dotto.FileUpload (S3 storage)
  └── Dotto.Ffmpeg (ffmpeg compression)
```

| Project | Purpose |
|---------|---------|
| **Dotto.Bot** | Entry point. `Startup.cs` bootstraps DI, NetCord, and hosted services. |
| **Dotto.Discord** | NetCord commands, event handlers, result handlers. |
| **Dotto.Application** | Business logic. |
| **Dotto.Common** | Shared utilities. No project dependencies. |
| **Dotto.Downloader.Contracts** | Pure interfaces: `IDownloaderService`, `DownloadedMedia`, `DownloaderType`. |
| **Dotto.Downloader** | `YtdlDownloaderService` (yt-dlp process) and `CobaltDownloaderService` (HTTP API). |
| **Dotto.Database** | EF Core + Npgsql. `DottoDbContext` with `ChannelFlags` and `DownloadedMedia` tables. |
| **Dotto.FileUpload** | S3 upload via `S3UploadService`. Optional — skipped if `Minio.BaseUrl` is null. |
| **Dotto.Ffmpeg.Contracts** | Pure interfaces: `IVideoCompressorStrategy`, `CompressionResult`, `CompressionMethod`, `CompressionOptions`. |
| **Dotto.Ffmpeg** | `FfmpegService` (ffmpeg process), `Vp9CompressionStrategy`, `Av1CompressionStrategy` (stub), `FfmpegTempCleanupService`. |
| **Dotto.Tests** | NUnit + NSubstitute + Shouldly + Testcontainers (PostgreSQL). |

### Bootstrap Sequence (`Startup.cs`)

1. `AddDatabase()` — Npgsql DbContext (skipped if connection string null)
2. `AddFileUploader()` — S3 client (skipped if `Minio.BaseUrl` null)
3. `AddDownloader()` — Ytdl (always) + Cobalt (only if `Downloader.Cobalt.BaseUrl` set)
4. `AddSingleton<IDateTimeProvider, DateTimeProvider>()` + `AddApplication()` — factories, HybridCache, services
5. `AddDiscordIntegration()` — `AutoDownloadSettings` options, command handlers, event processors
6. `AddHostedService<ChannelFlagPoller>()` — refreshes flag cache every 5 minutes
7. `Build()` → `MigrateDatabase()` → `InitializeS3Uploader()` (fire-and-forget) → `AddModules()` → `RunAsync()`

### DI Patterns

- **Keyed services**: Downloaders by `DownloaderType` via `AddKeyedSingleton`; compression strategies by `CompressionMethod` via `AddKeyedScoped`
- **Options**: `AddOptions<T>().BindConfiguration().ValidateDataAnnotations().ValidateOnStart()`
- **Scoped**: command handlers (transient), event processors, `ChannelFlagsService`, compression strategies
- **Singleton**: `DateTimeProvider`, `UrlCorrector`, `MediaProcessingService`, `FfmpegService`, downloader settings, resolved options values

## Commands

### Command Structure

Commands follow a strict separation: **command definitions** in `Dotto.Discord/Commands/` are thin invocation glue that delegate to **command handlers** in `Dotto.Discord/CommandHandlers/`. The same handler can be shared across slash commands, text commands, context menus, and auto-download.

#### Command Module Anatomy

```
Dotto.Discord/Commands/<Feature>/
  ├── ApplicationCommand.cs   ← Slash commands + context menus (ApplicationCommandModule<ApplicationCommandContext>)
  └── TextCommand.cs          ← Prefix text commands (CommandModule<CommandContext>)
```

Classes implementing CommandModule/ApplicationCommandModule/etc... are NOT manually registered in DI. NetCord discovers them via `host.AddModules(typeof(CommandAssemblyMarker).Assembly)` in `Startup.cs`.

### Adding a New Command

1. Create handler in `Dotto.Discord/CommandHandlers/<Feature>/`
2. Register as **transient** in `AddCommandHandlers()` in `Dotto.Discord/DependencyInjection.cs`
3. Create `ApplicationCommand.cs` and/or `TextCommand.cs` under `Dotto.Discord/Commands/<Feature>/`
4. Command modules are NOT registered in DI — NetCord discovers them via `host.AddModules(typeof(CommandAssemblyMarker).Assembly)`

### Generic Message Properties Pattern

Handlers use `T : IMessageProperties` to unify responses across command types:

- `InteractionMessageProperties` — slash commands (`RespondAsync`/`FollowupAsync`)
- `ReplyMessageProperties` — text commands (`ReplyAsync`)

Handler returns `Task<T>`, command module invokes with the appropriate type. Inject handlers via constructor parameters, not method parameters.

### Result Handlers


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [2048khz-gachi-rmx/dotto](https://github.com/2048khz-gachi-rmx/dotto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
