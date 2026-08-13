---
trigger: always_on
description: **Spotter** is an AI-powered playlist generator and metadata enrichment system for Navidrome. It aggregates listening history from multiple sources (Navidrome, Spotify, Last.fm), enriches music metadata using AI and external services, and generates intelligent playlists through customizable DJ personas.
---

# Spotter Development Guide

## Project Overview

**Spotter** is an AI-powered playlist generator and metadata enrichment system for Navidrome. It aggregates listening history from multiple sources (Navidrome, Spotify, Last.fm), enriches music metadata using AI and external services, and generates intelligent playlists through customizable DJ personas.

### Key Features
- Unified listening history across multiple music services
- AI-powered mixtape generation with customizable DJ personas
- Automatic metadata enrichment from MusicBrainz, Spotify, Last.fm, Fanart.tv, and OpenAI
- Playlist synchronization between services with intelligent track matching
- Real-time updates via Server-Sent Events (SSE)
- Retro-themed UI (1970s music cabinet light theme, 1980s cyberpunk dark theme)
- Pluggable provider and enricher architecture

### User Stories

**Authentication & Onboarding**
- As a user, I want to log in using my existing Navidrome credentials so I don't need another account
- As a user, I want to connect my Spotify account to import my listening history and playlists
- As a user, I want to connect my Last.fm account to sync my scrobble history
- As a user, I want to disconnect external services when I no longer want to share data

**Listening History**
- As a user, I want to see my recent listening history from all connected services in one unified view
- As a user, I want to paginate through my listening history to explore older listens
- As a user, I want to see real-time updates when new tracks are played without refreshing the page
- As a user, I want to see which service each listen came from (Navidrome, Spotify, Last.fm)

**Playlist Management**
- As a user, I want to view all my playlists from Navidrome, Spotify, and Last.fm in one place
- As a user, I want to sync playlists from Spotify or Last.fm to my Navidrome library
- As a user, I want to see sync status (pending, success, warning, error) for each playlist
- As a user, I want to see how many tracks were successfully matched during sync
- As a user, I want to manually trigger playlist sync when I make changes
- As a user, I want to rebuild a playlist sync from scratch if something goes wrong
- As a user, I want to disable sync for a playlist and optionally remove it from Navidrome
- As a user, I want to see detailed match statistics (total tracks, matched tracks, percentage)
- As a user, I want automatic periodic sync so my playlists stay up-to-date

**AI-Powered Mixtapes (Vibes Engine)**
- As a user, I want to create DJ personas with unique personalities and music preferences
- As a user, I want to generate mixtapes based on a DJ's personality and my listening history
- As a user, I want to seed mixtapes with specific artists, albums, or tracks
- As a user, I want to schedule mixtapes to regenerate daily, weekly, or monthly
- As a user, I want to see why the AI selected each track for my mixtape
- As a user, I want to sync generated mixtapes to Navidrome as playable playlists
- As a user, I want to enhance existing playlists with AI-suggested tracks that complement the vibe
- As a user, I want to reorder playlist tracks using AI for better flow and energy progression
- As a user, I want to create mixtapes inspired by specific artists from their detail pages

**Metadata Enrichment**
- As a user, I want artist biographies and tags automatically added to my library
- As a user, I want album summaries and metadata enriched from multiple sources
- As a user, I want high-quality artist and album images downloaded to my library
- As a user, I want AI-generated summaries and tags clearly marked in the UI
- As a user, I want to manually trigger enrichment for specific artists or albums
- As a user, I want periodic background enrichment so my library stays current

**Artist Discovery**
- As a user, I want to find similar artists within my own library using AI analysis
- As a user, I want to see confidence scores for artist similarities
- As a user, I want to see explanations for why artists are considered similar
- As a user, I want to refresh similar artist recommendations to discover new connections
- As a user, I want to create mixtapes inspired by artists I discover

**Preferences & Customization**
- As a user, I want to choose between light, dark, or system-based themes
- As a user, I want to customize the AI personality for playlist generation
- As a user, I want to configure pagination (items per page) for listings
- As a user, I want to see when my connected services last synced
- As a user, I want to manually trigger background tasks (sync, enrichment, cleanup)

**User Experience**
- As a user, I want toast notifications for important events (sync started, completed, failed)
- As a user, I want progress indicators during long-running operations
- As a user, I want timeago timestamps that automatically update (e.g., "5 minutes ago")
- As a user, I want keyboard navigation and accessibility features
- As a user, I want responsive design that works on mobile and desktop

### RFC 2119 Requirements


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joestump/spotter](https://github.com/joestump/spotter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
