---
trigger: always_on
description: This is the shared entry point for coding agents working in this repository. These instructions
---

# Coding Agent Instructions

This is the shared entry point for coding agents working in this repository. These instructions
apply throughout the repository, regardless of the agent or editor being used.

Before making changes, read and follow both shared rule documents:

- [Code style](docs/development/code-style.md)
- [Testing](docs/development/testing.md)

Keep project guidance in this file and the linked documentation. Any tool-specific instruction
files should only point here, so there is a single source of truth. For tools that do not discover
`AGENTS.md` automatically, explicitly include this file in their context or ask them to read it.

Historical design plans live in `docs/plans/`; consult them when relevant, but verify their
assumptions against the current code. The existing
[BitTorrent design assessment](docs/plans/kmp-expert-plan.md) is one such reference.

You are a senior Kotlin Multiplatform library engineer working on "Ketch", an open-source Kotlin
Multiplatform download manager library.

## Project Status

The library is **substantially complete** with all core features implemented and working across
Android, JVM/Desktop, iOS, and WebAssembly platforms.

**Note:** The library has not been published yet, so public API breaking changes are allowed.

## Module Structure

```
library/
  api/        # Public API interfaces and models -- published SDK module
  core/       # In-process download engine -- published SDK module
  ktor/       # Ktor-based HttpEngine implementation -- published SDK module
  ftp/        # FTP/FTPS DownloadSource (Android, iOS, JVM only) -- published SDK module
  torrent/    # BitTorrent/Magnet DownloadSource (Android, JVM, iOS) -- published SDK module
  kermit/     # Optional Kermit logging integration -- published SDK module
  sqlite/     # SQLite-backed TaskStore (Android, iOS, JVM only) -- published SDK module
  remote/     # Remote KetchApi client (HTTP + SSE) -- published SDK module
  endpoints/  # Shared REST API endpoint definitions (Ktor Resources)
config/       # Multiplatform TOML-based configuration (server, download, remotes)
server/       # Ktor-based daemon server with REST API and SSE events
ai/
  discover/   # LLM agent-driven resource discovery (JVM only, Koog framework)
app/
  shared/     # Shared Compose Multiplatform UI (supports Core + Remote backends)
  android/    # Android app
  desktop/    # Desktop (JVM) app
  web/        # Wasm browser app
  ios/        # Native iOS app (Xcode project, consumes shared module)
cli/          # JVM CLI entry point
```

## Package Structure

### `library:api` (public API)
- `com.linroid.ketch.api` -- `KetchApi`, `DownloadTask`, `DownloadRequest`, `DownloadState`,
  `DownloadProgress`, `Segment`, `KetchError`, `SpeedLimit`, `DownloadPriority`,
  `DownloadSchedule`, `DownloadCondition`, `KetchVersion`, `ResolvedSource`, `SourceFile`,
  `FileSelectionMode`
- `com.linroid.ketch.api.config` -- `DownloadConfig`

### `library:core` (implementation)
- `com.linroid.ketch.core` -- `Ketch` (implements `KetchApi`)
- `com.linroid.ketch.core.engine` -- `HttpEngine`, `DownloadCoordinator`, `RangeSupportDetector`,
  `ServerInfo`, `DownloadSource`, `HttpDownloadSource`, `SourceResolver`, `SourceInfo`,
  `SourceResumeState`, `DownloadContext`, `DownloadQueue`, `DownloadScheduler`,
  `SpeedLimiter`, `TokenBucket`, `DelegatingSpeedLimiter`
- `com.linroid.ketch.core.segment` -- `SegmentCalculator`, `SegmentDownloader`
- `com.linroid.ketch.core.file` -- `FileAccessor`, `PathFileAccessor`, `NoOpFileAccessor`,
  `PlatformFileSystem` (expect/actual), `FileNameResolver`, `DefaultFileNameResolver`,
  `PathSerializer`
- `com.linroid.ketch.core.log` -- `Logger`, `KetchLogger`
- `com.linroid.ketch.core.task` -- `RealDownloadTask`, `TaskStore`, `InMemoryTaskStore`,
  `TaskRecord`, `TaskState`

### `library:ftp`
- `com.linroid.ketch.ftp` -- `FtpDownloadSource` (implements `DownloadSource`), `FtpClient`,
  `RealFtpClient`, `FtpUrl`, `FtpReply`, `FtpError`, `FtpResumeState`, `TlsUpgrade`

### `library:torrent`
- `com.linroid.ketch.torrent` -- `TorrentDownloadSource` (implements `DownloadSource`),
  `TorrentEngine`, `TorrentSession`, `TorrentConfig`, `TorrentMetadata`,
  `TorrentResumeState`, `MagnetUri`, `InfoHash`, `Bencode`, `Sha1`

### `library:endpoints`
- `com.linroid.ketch.endpoints` -- `Api` (Ktor `@Resource` definitions for REST API)
- `com.linroid.ketch.endpoints.model` -- `TaskSnapshot`, `TasksResponse`, `TaskEvent`,
  `TaskEventType`, `ErrorResponse`, `ResolveUrlRequest`, `SpeedLimitRequest`,
  `PriorityRequest`, `ConnectionsRequest`

### `config`
- `com.linroid.ketch.config` -- `KetchConfig`, `ConfigStore`, `FileConfigStore`,
  `ServerConfig`, `RemoteConfig`, `AiSettings`, `LlmSettings`, `LlmProvider`,
  `SearchSettings`, `SearchProvider`, `PlatformFileSystem` (expect/actual)

### `library:remote`
- `com.linroid.ketch.remote` -- `RemoteKetch` (implements `KetchApi`), `RemoteDownloadTask`,
  `ConnectionState`, `WireModels`, `WireMapper`

### `ai:discover` (JVM/Android only)
- `com.linroid.ketch.ai` -- `AiModule`, `AiConfig`, `LlmClientFactory`,
  `ResourceDiscoveryService`, `DiscoverQuery`, `DiscoverResult`,
  `RankedCandidate`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [linroid/Ketch](https://github.com/linroid/Ketch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
