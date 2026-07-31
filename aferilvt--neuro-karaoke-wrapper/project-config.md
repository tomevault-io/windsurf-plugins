---
trigger: always_on
description: This document provides a comprehensive overview of the Neuro Karaoke Android application, including its structure, key technologies, and instructions on how to build and run the app.
---

# Neuro Karaoke Project Overview

This document provides a comprehensive overview of the Neuro Karaoke Android application, including its structure, key technologies, and instructions on how to build and run the app.

## High-Level Description

The Neuro Karaoke app is a music player application that allows users to browse and play karaoke songs from the Neuro Karaoke website (neurokaraoke.com). It features a modern user interface built with Jetpack Compose and uses ExoPlayer (Media3) for audio playback with media notification support. The app connects to the Neuro Karaoke API to fetch playlists and songs.

## Package Name

`com.soul.neurokaraoke`

## Project Structure

The project is a single-module Android application with the following structure:

-   **`app` module:** The main application module.
    -   **`src/main/java/com/soul/neurokaraoke`:** The root package for the application's source code.
        -   **`MainActivity.kt`:** The main entry point of the application.
        -   **`data`:** Contains the data layer of the application.
            -   **`api/NeuroKaraokeApi.kt`:** Handles communication with the Neuro Karaoke API.
            -   **`api/LyricsApi.kt`:** Fetches lyrics from lrclib.net API.
            -   **`model/`:** Defines the data models:
                -   `Song.kt` - Song data with Singer enum (NEURO, EVIL, DUET, OTHER)
                -   `Playlist.kt` - Playlist with id, title, coverUrl, previewCovers
                -   `User.kt` - Discord user data for authentication
            -   **`repository/`:** Manages data sources:
                -   `SongRepository.kt` - Song data management
                -   `AuthRepository.kt` - Discord OAuth authentication
                -   `UserPlaylistRepository.kt` - User-created playlists (local storage)
                -   `ArtistImageRepository.kt` - Maps artist names to Last.fm image URLs
            -   **`PlaylistCatalog.kt`:** Manages local playlist catalog.
        -   **`audio`:** Audio processing components.
            -   **`AudioCacheManager.kt`:** Manages 500MB audio cache for offline/smooth playback.
            -   **`EqualizerManager.kt`:** Equalizer and bass boost controls.
        -   **`navigation`:** Defines the navigation graph and screens.
        -   **`service`:**
            -   **`MediaPlaybackService.kt`:** Background media service with notification support and audio caching.
        -   **`ui`:** Contains the UI layer.
            -   **`MainScreen.kt`:** Main screen with navigation drawer, scaffold, and mini player.
            -   **`components/`:** Reusable UI components.
                -   `AddToPlaylistSheet.kt` - Bottom sheet for adding songs to user playlists
            -   **`screens/`:** The different screens:
                -   `home/HomeScreen.kt` - Main home with Cover Distribution & Top Genres
                -   `setlist/SetlistScreen.kt` - Playlist grid with 2x2 cover previews
                -   `setlist/SetlistDetailScreen.kt` - Detailed playlist view (website-style)
                -   `player/PlayerScreen.kt` - Full player with lyrics, equalizer, queue
                -   `search/SearchScreen.kt` - Search across ALL playlists
                -   `library/FavoritesScreen.kt` - Favorites with Discord sign-in
                -   `library/PlaylistsScreen.kt` - User-created playlists
                -   `library/UserPlaylistDetailScreen.kt` - User playlist detail with Play/Shuffle/Download All
            -   **`theme/`:** Theme colors and styling (Neuro/Evil themes).
        -   **`viewmodel/`:**
            -   `PlayerViewModel.kt` - Manages player state via MediaController
            -   `AuthViewModel.kt` - Manages Discord authentication

## Key Technologies and Libraries

-   **Kotlin:** The primary programming language
-   **Jetpack Compose:** Modern Android UI toolkit
-   **ExoPlayer (Media3):** Audio playback with MediaSession support
-   **MediaSessionService:** Background playback with notification controls
-   **Media3 DataSource:** Audio caching with SimpleCache and CacheDataSource
-   **Coil:** Image loading library
-   **Jetpack Navigation:** Screen navigation
-   **Coroutines and Flow:** Async programming
-   **Android AudioFX:** Equalizer and BassBoost effects

## API Integration

- **Base URL:** `https://idk.neurokaraoke.com`
- **API URL:** `https://api.neurokaraoke.com`
- **Endpoints:**
  - `GET /public/playlist/{playlistId}` (base URL) - Returns playlist info and songs
  - `GET /api/playlists?startIndex=0&pageSize=200&isSetlist=True&year=0` (API URL) - Official setlists
  - `GET /api/playlist/public` (API URL) - Public playlists
  - `GET /api/stats/cover-distribution` (API URL) - Cover distribution stats (totalSongs, neuroCount, evilCount, duetCount, otherCount)
- **Storage URL:** `https://storage.neurokaraoke.com` - Audio and images
- **Lyrics API (primary):** `GET /api/songs/{songId}/lyrics` (API URL) - Synced lyrics as `[{time, text}]` array
- **Lyrics API (fallback):** `https://lrclib.net/api/search` - Synced lyrics fetching

## Features Implemented

### 1. Search All Songs
- Search screen loads songs from ALL playlists (not just current)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AferilVT/neuro-karaoke-wrapper](https://github.com/AferilVT/neuro-karaoke-wrapper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
