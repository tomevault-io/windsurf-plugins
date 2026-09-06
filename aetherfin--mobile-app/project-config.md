---
trigger: always_on
description: This document is the **single source of truth** for AI coding agents working
---

# CLAUDE.md — Aetherfin Project Guide

This document is the **single source of truth** for AI coding agents working
on Aetherfin. Read this end to end before making changes. It captures the
constraints, conventions, and gotchas discovered while building the client
and is updated as we learn more.

## 1. What Aetherfin is

A native-feeling Android music player that operates in one of two modes:
- **Server mode** — streams from a self-hosted **Jellyfin** or **Navidrome** server.
- **Local mode** — plays audio files from device storage via SAF (Storage Access Framework).

The only first-class platform is Android. iOS may follow but is not
considered when making trade-offs.

> Aetherfin must read as a *premium music* app,
> not "a Jellyfin/Navidrome client."

### 1.1 App modes

The user picks a mode during onboarding. Only one mode is active at a time.
Mode is persisted in `shared_preferences` via `AppModeStore`. Switching
mode returns to onboarding.

```dart
enum AppMode { server, local }
```

### 1.2 Mental model — what runs where (Server mode)

Treat the server (Jellyfin or Navidrome) as a **file source + library state
store**, nothing else. **Aetherfin is the player.**

The app supports two server backends via the `MusicBackend` abstraction
(`lib/core/backend/music_backend.dart`). `ServerType` (jellyfin | subsonic)
is persisted in auth storage and determines which client is instantiated.

**Server (Jellyfin or Navidrome) is responsible for:**
- Storing audio files and serving the original bytes byte-for-byte.
- Catalog metadata: titles, artists, albums, genres, durations, year, artwork.
- Search (Jellyfin: `/Users/{id}/Items?searchTerm=…` — never `/Search/Hints`;
  Navidrome: `search3.view`).
- Auth (Jellyfin: user accounts + access tokens; Navidrome: Subsonic token
  auth — `md5(password + salt)`).
- Per-user state: favorites, play counts, last-played-at, playlists.
- LRC lyric files (stored only; the app parses them).
- "Now Playing" telemetry display — Jellyfin: `POST /Sessions/Playing*`;
  Navidrome: `scrobble.view`.

### 1.3 Mental model — Local mode

In local mode there is no server. The app scans device folders via SAF,
extracts metadata using Android's `MediaMetadataRetriever`, caches it in
a local SQLite database (`sqflite`), and plays files via `content://` URIs.

**Local mode provides:**
- Albums, Artists, Songs, Genres (parsed from file tags)
- Cover art (embedded in files, cached to disk)
- Full playback: queue, shuffle, loop, gapless, visualizer, EQ/DSP
- Favorites and playlists (stored in local SQLite via Drift)
- Smart playlists (resolved via SQL against the local tag cache)

**Aetherfin (app) is responsible for everything, on-device:**
- All audio **decoding** (libmpv via `mpv_audio_kit`).
- Buffering, gapless transitions, output routing, position tracking.
- Queue management (order, shuffle, repeat, reorder).
- UI rendering for every screen.
- LRC parsing + synced line highlighting.
- Real-time FFT spectrum + client-side DSP for the visualizer.
- FFT-driven artwork pulse (sub-bass transient detector).
- Spectral color extraction from artwork (`palette_generator_master`).
- Lock-screen / notification media-session integration (custom Kotlin `MediaSessionCompat` + MethodChannel).
- Cover-art file cache (`cached_network_image` for server, `Image.file` for local).
- Local settings: audio-quality preference, sleep timer, EQ/DSP, ReplayGain.
- **Local mode only:** metadata scanning, SQLite cache, SAF folder management.

**Strict consequence (Jellyfin):** stream URLs MUST use
`/Audio/{id}/stream?Static=true` — the direct-stream endpoint. The
universal endpoint (`/Audio/{id}/universal`) triggers server-side
transcoding to HLS, which (1) wastes the server's CPU, (2) gives a
codec that may not decode cleanly, and (3) was the cause of the
"track plays but position never advances" bug.

**Strict consequence (Navidrome):** stream URLs use
`/rest/stream.view?id={id}&u=…&t=…&s=…&v=1.16.1&c=Aetherfin&f=json`.
Auth is embedded as query parameters (username, token, salt) because
libmpv/FFmpeg cannot use Subsonic header auth.

**Strict consequence #2:** stream URLs embed auth as query parameters.
Jellyfin: `api_key=<token>`. Navidrome: `u`, `t` (md5 hash), `s` (salt).
FFmpeg/libmpv rejects the `Authorization: MediaBrowser …` header
(commas treated as header-list separators).

**Strict consequence #3:** favorites, play counts, and playlist contents
are server-owned even though the heart icon flips locally first.
Jellyfin: `POST/DELETE /Users/{id}/FavoriteItems/{itemId}`.
Navidrome: `star.view` / `unstar.view`.
On HTTP error, revert. Never store favorite state only on-device.

## 2. Tech stack (exact versions)

| Layer | Choice | Notes |
|---|---|---|
| Framework | Flutter **3.44.0 stable**, Dart **3.11.5** | `flutter --version` must match. CI pins this. |
| State | `flutter_riverpod` ^2.6 | `FutureProvider.autoDispose`, `StateNotifierProvider`. No `ChangeNotifier` for Riverpod providers. |
| Routing | `go_router` ^14.7 | Shell route with bottom nav. See `lib/app/router.dart`. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Aetherfin/mobile-app](https://github.com/Aetherfin/mobile-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
