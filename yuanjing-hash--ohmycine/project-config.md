---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project status

OhMyCine is currently in the requirements analysis and architecture design phase. The repository may contain product and design documentation before runnable component code exists. Treat README quick-start snippets and planned directories as target design unless the corresponding files actually exist.

When reviewing or editing documentation, do not interpret phased roadmap language as cutting scope. The intended product scope is complete; roadmap changes should adjust implementation order and dependencies, not remove planned capabilities.

## Product understanding

OhMyCine is an open-source, self-hosted, cross-platform home cinema ecosystem. It is not just a player and not just a media automation backend. The final system combines:

- immersive media playback
- media source browsing
- local and cloud-drive playback
- metadata scraping and poster-wall library views
- STRM generation
- 302 direct-link playback
- PT resource discovery
- automated downloading
- automated transfer and library organization
- Emby/Jellyfin refresh notifications
- TV follow/subscribe automation
- AI recommendations based on the user's own library
- plugin distribution
- CLI automation and diagnostics

Core media pipeline:

```text
Discover → Download → Transfer → Import → Notify
```

Typical full flow:

```text
PT/search result → downloader → category rule → storage destination → STRM/302 → Emby/Jellyfin refresh → Player display
```

Existing-media flow:

```text
Local / Emby / Jellyfin / OpenList / CloudDrive2 / 115 → Player browse → scrape metadata if needed → play → recommend
```

## Component responsibilities

### Player

Player is the primary user-facing application and must be independently useful without Server. It should be developed first as a complete client, with Server-related surfaces left as integration entry points when Server functionality is unavailable.

Player responsibilities:

- play local and remote video using Tauri + Vue + Rust + libmpv
- provide Cinema OS style UI: dark theme, liquid glass, poster wall, hero banner, continue watching, animations
- support local files, drag-and-drop playback, file association, playlists, playback history, subtitles, audio tracks, keyboard shortcuts
- implement a DataSource abstraction so all media sources expose common operations: list, search, getDetail, getStreamURL
- connect directly to Emby/Jellyfin via native APIs
- connect directly to OpenList/Alist and CloudDrive2/WebDAV
- support local-file media sources
- reserve future DataSources such as 115, 123, Quark, and ServerDataSource
- scrape metadata locally for file/cloud sources that lack media metadata: filename parse → TMDB lookup → local SQLite/cache → poster wall
- implement AI recommendation on the Player side using the user's API key and local media metadata/RAG; it should recommend only media the user already has
- connect to Server for enhanced features such as downloads, discovery, follow tasks, sync, and real-time events, but not depend on it for basic playback

Playback architecture preference:

- The product direction favors libmpv embedding through Rust FFI over an MPV sidecar because the goal is a deeply integrated, immersive player UI.
- MPV sidecar is easier and faster to validate but has weaker UI/video integration.
- libmpv embedding has a higher product ceiling but requires careful platform-specific rendering, packaging, and mobile strategy.

### Server

Server is the self-hosted automation engine. It is not merely an API backend for Player; it owns the media pipeline.

Server's core abstraction is the three-layer storage model:

```text
Connections → Storage Destinations → Category Rules
```

- Connections answer: what external service can OhMyCine connect to? Examples: Emby, Jellyfin, OpenList/Alist, CloudDrive2, 115, local filesystem, qBittorrent, Transmission, PT sites.
- Storage Destinations answer: where should files ultimately live? Examples: movie library on OpenList, TV library on 115, documentary folder on local NAS.
- Category Rules answer: what content type should go to which destination, with which naming template and transfer strategy?

Server responsibilities:

- manage media server, cloud drive, local file, downloader, and PT-site connections
- support the initial must-have storage set: 115, OpenList/Alist, CloudDrive2, and local files
- manage storage destinations and classification rules
- control qBittorrent/Transmission downloads
- implement STRM generation, incremental sync, full sync, invalid STRM cleanup, NFO/poster generation
- implement 302 proxy/direct-link playback for cloud media
- notify Emby/Jellyfin to refresh libraries
- expose REST API and WebSocket event streams
- support PT aggregation, one-click download, and metadata matching
- support follow/subscribe tasks for TV series: search missing episodes, apply quality/group filters, and submit downloads
- support multi-user permissions eventually; do not remove this scope from docs, but it can be phased after core flows
- support plugins eventually through Hub integration


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yuanjing-hash/OhMyCine](https://github.com/yuanjing-hash/OhMyCine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
