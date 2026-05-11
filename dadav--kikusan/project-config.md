---
trigger: always_on
description: Kikusan is a tool to search and download music from youtube music. It must use yt-dlp in the background. It must be usable through CLI and also have a web app (subcommand "web"). The web app should be really simple, but must support search functionality. It should be deployable with docker and have an example docker-compose file. It must add lyrics via lrc files to the downloaded files (via https://lrclib.net/).
---

# Project description

Kikusan is a tool to search and download music from youtube music. It must use yt-dlp in the background. It must be usable through CLI and also have a web app (subcommand "web"). The web app should be really simple, but must support search functionality. It should be deployable with docker and have an example docker-compose file. It must add lyrics via lrc files to the downloaded files (via https://lrclib.net/).

## Features

### Web UI

- Search functionality with results display
  - **URL Support**: Paste YouTube Music URLs directly into search bar
    - Single track URLs: `https://music.youtube.com/watch?v=VIDEO_ID`
    - Playlist URLs: `https://music.youtube.com/playlist?list=PLAYLIST_ID`
    - Also supports `youtube.com` and `youtu.be` URL variants
    - Playlist URLs return ALL tracks (no 20-result limit like text search)
    - Results displayed in same format as text search
    - **Radio playlist handling**: Radio playlists (IDs starting with `RDAM`) are not supported due to different API structure
      - URLs with both video and radio playlist (e.g., `?v=VIDEO_ID&list=RDAM...`) fall back to returning just the video
      - Radio-only playlist URLs return clear error: "Radio playlists are not supported"
      - Protection implemented in both `parse_youtube_url()` and `get_playlist_tracks()`
  - Backend auto-detects URL vs text query using `parse_youtube_url()` in `search.py`
  - URL metadata fetched via ytmusicapi: `get_track_from_video_id()` for singles, `get_playlist_tracks()` for playlists
  - **Deezer playlist URL support**: `https://www.deezer.com/playlist/{id}` (also localized paths like `/us/playlist/{id}`)
    - Deezer tracks are resolved to YouTube Music via search (`"{title} {artist}"`) so the UI can queue/download them through existing `video_id` flow
    - Implemented in `api_search()` using `kikusan/deezer.py`
  - **Playlist progress streaming (web UI)**:
    - Playlist searches use Server-Sent Events to show progress (matching `x/y` tracks for Deezer, resolved count for YouTube playlists)
    - Endpoint: `GET /api/search/playlist/stream?q=...` in `kikusan/web/app.py`
    - Frontend uses `EventSource` with `progress`, `complete`, and `failure` events in `kikusan/web/templates/index.html`
    - Test coverage: `tests/test_web_search_playlist_stream.py`
- View counts displayed for each song (e.g., "1.9B views", "47M views")
  - View counts are retrieved from ytmusicapi search results (no additional API calls needed)
  - Displayed alongside duration in the track metadata section
- Download button for each track
- Dark/light theme toggle
- Version display in header (dynamically loaded from `pyproject.toml` via `importlib.metadata`)
- **Explore tab**: Browse moods, genres, and music charts from YouTube Music
  - Three-level navigation: Categories -> Playlists -> Tracks (with breadcrumb nav)
  - Mood/genre categories displayed as clickable grid cards
  - Charts with country selector (11 countries + global)
  - **New Releases**: Grid of newly released albums from YouTube Music explore page
    - Album cards with cover art, title, artist, type (Album/Single/EP), year, and explicit badge
    - "View Tracks" button opens album tracks in Songs tab (reuses existing album view flow)
    - "Download Album" button queues all album tracks for download
    - Paginated with 8 albums per page
    - "+ Cron" button generates `type: new_releases` cron.yaml config snippet
    - Data fetched via `/api/explore/new-releases` endpoint (30 min cache)
    - Uses ytmusicapi `get_explore()` → `new_releases` section
    - CSS classes: `.explore-new-releases-section`, `.new-releases-grid`, `.new-release-card`, `.explicit-badge`
  - View counts displayed for chart tracks and playlist tracks (when available from API)
  - Play preview button and Copy URL button on all explore track listings (charts and playlists)
  - Duration displayed alongside view counts in chart track metadata
  - "Download All" buttons for bulk queueing of playlist tracks and chart tracks
  - "Cron" button (+ Cron) on charts header, new releases header, and mood/genre breadcrumb for copying cron.yaml config snippets to clipboard
    - Charts: generates `type: charts` config with selected country, sync, schedule, and limit
    - New Releases: generates `type: new_releases` config with sync, schedule, and limit
    - Moods: generates `type: mood` config with category params, sync, and schedule
    - Sanitizes category titles into valid cron config keys (lowercase, alphanumeric + dashes)
    - Clipboard copy with "Copied!" feedback and fallback for older browsers
    - CSS: `.cron-btn` (standard) and `.cron-btn-inline` (breadcrumb variant) in `style.css`
    - JS: `generateCronYaml()` and `copyCronConfig()` in embedded script
  - Reuses existing download queue infrastructure (`/api/queue/add`)
- **Downloads tab**: View and manage the M3U download playlist from the web UI

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dadav/kikusan](https://github.com/dadav/kikusan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
