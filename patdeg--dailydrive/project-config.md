---
trigger: always_on
description: This file helps AI coding assistants (Claude Code, Gemini, Copilot, etc.) understand and work with this project. If a user asks you to help them set up Daily Drive, follow the setup workflow below.
---

# Daily Drive — AI Assistant Guide

This file helps AI coding assistants (Claude Code, Gemini, Copilot, etc.) understand and work with this project. If a user asks you to help them set up Daily Drive, follow the setup workflow below.

## What This Project Does

Recreates Spotify's discontinued "Daily Drive" feature — a playlist that mixes podcast episodes and music tracks, updated automatically on a schedule. Runs on Raspberry Pi, Orange Pi, or any Linux machine.

## Tech Stack

- **Runtime:** Node.js (v18+)
- **Spotify Library:** `spotify-web-api-node` — wraps the Spotify Web API
- **Config:** YAML via `js-yaml`
- **Auth:** OAuth 2.0 Authorization Code flow with token persistence
- **Scheduling:** systemd timer or cron

## Project Structure

```
index.js              — Main script: fetches podcasts + music, mixes, updates playlist
setup.js              — One-time OAuth setup: starts local server, catches callback, saves token
taste-profile.js      — LLM-powered genre detection via Demeterics API
config.example.yaml   — Config template with comments explaining every field
config.yaml           — User's actual config (git-ignored, contains secrets)
.env                  — API keys for Demeterics etc. (git-ignored)
.spotify-token.json   — Saved OAuth tokens (git-ignored, auto-refreshed)
state.json            — Run state cache (git-ignored, tracks last episode URIs)
install.sh            — Quick installer for fresh Linux machines
systemd/              — Service + timer files for auto-scheduling
package.json          — Dependencies and npm scripts
.gitignore            — Comprehensive protection for secrets (PUBLIC REPO)
```

## Key Commands

```bash
npm install           # Install dependencies
npm run setup         # One-time Spotify authentication (deletes old token first)
npm start             # Run the playlist builder
npm test              # Dry run (shows what would happen without changing the playlist)
npm run taste         # Auto-detect genre tags via LLM (requires DEMETERICS_API_KEY in .env)
```

## Setup Workflow (for AI assistants helping users)

When a user asks you to help set up Daily Drive, follow these steps:

### 1. Spotify App Creation
Guide the user to https://developer.spotify.com/dashboard to create an app:
- **Redirect URI:** `http://127.0.0.1:8888/callback` (NOT localhost — removed Nov 2025)
- **APIs:** Check both **Web API** and **Web Playback SDK**
- **User Management:** User MUST add their Spotify email in Settings > User Management (even as app owner). Without this, playlist writes return 403 Forbidden.

### 2. Create config.yaml
Copy from `config.example.yaml` and fill in:
- `spotify.client_id` and `spotify.client_secret` from the Dashboard
- `playlist_id` — user creates an empty playlist in Spotify, shares link, extract ID
- `podcasts` — user shares Spotify show links, extract IDs
- `music` — configure top_tracks, genres, and/or source playlists

### 3. OAuth Authentication
- If on SSH/headless: user needs SSH tunnel: `ssh -L 8888:127.0.0.1:8888 user@server`
- Run `npm run setup` — it deletes any old token and starts fresh OAuth flow
- User opens the printed URL in their local browser and approves
- Token saved to `.spotify-token.json`

### 4. Test and Run
- `npm test` — dry run to verify everything works
- `npm start` — actually update the playlist
- If 403 Forbidden: check User Management in Dashboard, re-run `npm run setup`

## IMPORTANT: Security (Public Repo)

This is a PUBLIC repository. The `.gitignore` is comprehensive but verify:
- **NEVER** commit `config.yaml` (contains client_id, client_secret)
- **NEVER** commit `.spotify-token.json` (contains access/refresh tokens)
- **NEVER** commit `.env` or any `*credentials*` / `*secret*` files
- **NEVER** commit `state.json` (runtime data)
- Before any commit, run `git status` and verify no sensitive files are staged
- If a user pastes credentials in chat, remind them this goes in config.yaml (git-ignored), not in any tracked file

## How the Code Works

### Authentication Flow (setup.js)
1. Deletes any existing `.spotify-token.json` for clean auth
2. Reads Spotify credentials from `config.yaml`
3. Starts Express server on `127.0.0.1:8888`
4. Generates Spotify auth URL with required scopes
5. User approves in browser → Spotify redirects with auth code
6. Exchanges code for access + refresh tokens
7. Saves tokens to `.spotify-token.json`

### Required Scopes
```
playlist-modify-public
playlist-modify-private
playlist-read-private
playlist-read-collaborative
user-library-read
user-read-private
user-read-recently-played
user-top-read
```

### Playlist Building Flow (index.js)
1. Loads config and tokens
2. Auto-refreshes access token if expiring within 5 minutes
3. Fetches latest episodes for each podcast via `getShowEpisodes()`
4. Checks state cache — skips update if episodes haven't changed
5. Fetches music from top tracks, source playlists, and/or genre search
6. Separates pinned episodes (`position: first`) from mixable episodes
7. Places pinned episodes first, then interleaves rest using `mix_pattern`
8. Replaces playlist content via `PUT /v1/playlists/{id}/items`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [patdeg/dailydrive](https://github.com/patdeg/dailydrive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
