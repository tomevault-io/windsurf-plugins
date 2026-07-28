---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Youtarr is a Dockerized application that automatically downloads videos from YouTube channels and integrates them with Plex Media Server. It consists of:

- **Backend**: Node.js/Express server with MariaDB database using Sequelize ORM
- **Frontend**: React/TypeScript application with a custom UI layer (Radix primitives + Tailwind CSS, themed via CSS variables)
- **Infrastructure**: Docker Compose setup with separate containers for app and database

## Scope Discipline

**IMPORTANT: Change only what the task requires.** This rule overrides every other rule in this file when they conflict.

- **Bug fixes**: make the minimum change needed to fix the bug. Do not rewrite surrounding code for style, extract helpers you don't need for the fix, or clean up while you're there.
- **Features**: touch only files the feature requires. Write new code to the standards below. Leave pre-existing code alone unless the feature forces you to change it.
- **Refactors**: only when the user explicitly asks for a refactor, and only as wide as the user scopes it.
- **Unrelated issues you notice** (oversized files, duplication, legacy patterns, missing tests, `console.log` calls, `any` types, drifted error-response shapes): list them in your final report so the user can decide whether to address them. **Do not fix them.**
- **When in doubt**: ask. A small clarifying question is cheaper than an unwanted refactor.

The size and quality rules in this file apply to code you are **creating or substantially rewriting**. They are not a mandate to refactor existing code you pass through.

## Task Handling

For multi-part requests (e.g., "review this PR AND explain WebSocket handling"), explicitly acknowledge all parts and complete each one before considering the task done.

## Architecture

### Backend (server/)
- `server.js`: Express entry point. `db.js`: Sequelize setup. `logger.js`: Pino logger with request correlation.
- `models/`: Sequelize models (channel, video, job, jobvideo, jobvideodownload, channelvideo, session, apikey, playlist, playlistvideo, playlistsyncstate, subfolder, videowatchstatus). Associations: Channel hasMany Videos, Job hasMany JobVideos, Playlist hasMany PlaylistVideos and hasMany PlaylistSyncStates.
- `routes/`: API handlers (auth, channels, videos, videoDetail, videoSearch, channelSearch, config, jobs, plex, setup, subscriptions, apikeys, ytdlpOptions, health, maintenance, playlists, mediaServers, subfolders). All use the dependency injection factory pattern; wiring lives in `server/routes/index.js`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DialmasterOrg/Youtarr](https://github.com/DialmasterOrg/Youtarr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
