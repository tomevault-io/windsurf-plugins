---
trigger: always_on
description: > Read this first. It is the high-signal map of the repo. For day-to-day code-style rules and
---

# Downloader — Codebase Guide

> Read this first. It is the high-signal map of the repo. For day-to-day code-style rules and
> token-efficient navigation, also see the **`code-navigator`** skill in `.claude/skills/`.

## What this project is

**Downloader** (`bezzad/Downloader`) is a .NET NuGet library (currently **v5.8.0**) for fast,
cross-platform multipart file downloads. It supports parallel chunked downloads, pause/resume,
bandwidth throttling, progress events, custom `HttpClient`/handler injection, and Native AOT.
Targets **.NET 8, 9, and 10**. The library has **no external runtime dependencies** and is
`IsAotCompatible = true` (System.Text.Json source-gen, no reflection serialization).

---

## Repository layout

```
src/
  Downloader/                  # Main library (the NuGet package)
    Exceptions/                # IncompleteDownloadException, FileExistException
    Extensions/                # ExceptionHelper, FileHelper, UrlHelper
    Helpers/                   # internal helpers
  Downloader.Test/             # All tests (xUnit)
    UnitTests/                 # Component-level tests
    IntegrationTests/          # End-to-end tests against DummyHttpServer
    HelperTests/               # Utility/server-endpoint tests
  Downloader.DummyHttpServer/  # ASP.NET test server used by integration tests
  Samples/Downloader.Sample/   # Console sample app (download.json drives URLs)
  .editorconfig                # Authoritative code-style rules (see skill)
```
`.editorconfig` lives in `src/`, not the repo root.

---

## Key source files (`src/Downloader/`)

| File | Role |
|---|---|
| `AbstractDownloadService.cs` | Base class: lifecycle, events, pause/resume/cancel, progress aggregation, `Serializer` |
| `DownloadService.cs` | Concrete impl: `StartDownload()`, parallel/serial dispatch, auto-resume, terminal-state rule |
| `DownloadConfiguration.cs` | All settings; `INotifyPropertyChanged`, `ICloneable` |
| `RequestConfiguration.cs` | Per-request HTTP knobs: headers, proxy, auth, cookies, redirect, user-agent |
| `Request.cs` | HTTP request model; URL→filename extraction |
| `SocketClient.cs` | `HttpClient`/`SocketsHttpHandler` wrapper: header fetch, range-support probe, **redirect handling**, file-size + filename extraction |
| `DownloadPackage.cs` | Live state snapshot: chunks, file size, status, storage handle; `TrySetCompleteState()` |
| `Chunk.cs` | One file segment: start/end/position, retry counter, timeout |
| `ChunkHub.cs` | Splits the file into N chunks honoring `MinimumChunkSize` |
| `ChunkDownloader.cs` | Downloads one chunk: retry/backoff loop, pause support, `ArrayPool` reads, throttle wiring |
| `ConcurrentStream.cs` | Thread-safe write stream with a background `Watcher` task |
| `ConcurrentPacketBuffer.cs` | Bounded async queue of `Packet`s (back-pressure for memory cap) |
| `ThrottledStream.cs` | Enforces per-chunk bandwidth limit |
| `Bandwidth.cs` | Tracks instantaneous + average speed |
| `PauseTokenSource.cs` / `PauseToken.cs` | Pause primitive over `SemaphoreSlim` |
| `DownloadBuilder.cs` | Fluent API: `DownloadBuilder.New().WithUrl(...).Build().StartAsync()` |
| `Extensions/ExceptionHelper.cs` | `IsMomentumError` (retry classification), `IsRedirectError`, cert validation |
| `Extensions/UrlHelper.cs` | `EnsurePathEncoded` — normalizes attacker-influenceable URLs before `new Uri()` |

---

## Architecture overview

### Download flow

```
DownloadFileTaskAsync(url, path)
  └── InitialDownloader()            // SocketClient, Request, ChunkHub, semaphores
  └── StartDownload()
        ├── GetFileSizeAsync()       // probe via SocketClient (Range: 0-0)
        ├── IsSupportDownloadInRange()
        ├── ProvideDownloadOnFile()
        │     └── TryResumeFromExistingFile()   // read metadata appended to .download file
        ├── Package.BuildStorage()   // ConcurrentStream over FileStream or MemoryStream
        ├── ChunkHub.SetFileChunks()
        ├── ParallelDownload() | SerialDownload()
        │     └── DownloadChunk() × N → ChunkDownloader.Download()
        │           └── ReadStream() → ConcurrentStream.WriteAsync()
        └── SendDownloadCompletionSignal()
              └── Package.TrySetCompleteState()  // truncate metadata, rename .download → final
```

### Producer–consumer for writes

`ChunkDownloader` rents buffers from `ArrayPool<byte>.Shared`, wraps them in `Packet`s (buffer +
position) and pushes to `ConcurrentPacketBuffer`. A single `LongRunning` `Watcher` task in
`ConcurrentStream` dequeues packets, writes them to the underlying stream in order, and returns the
buffers. `MaximumMemoryBufferBytes` bounds the queue (back-pressure).

### Auto-resume (`.download` files)

The library **always** downloads to a temp file with `DownloadFileExtension` (default `.download`),
then renames on success — independent of `EnableAutoResumeDownload`.

When `EnableAutoResumeDownload = true`:
- During download (~1 s cadence) `Package` state is serialized (JSON→binary) and **appended after**
  `TotalFileSize` inside the `.download` file.
- On resume: bytes past `TotalFileSize` = metadata → deserialize, verify `TotalFileSize` matches the
  server, restore `Chunks[]`. Any mismatch → discard and download fresh.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bezzad/Downloader](https://github.com/bezzad/Downloader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
