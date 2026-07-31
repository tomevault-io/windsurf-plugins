---
trigger: always_on
description: Enables remote control across devices. Devices join rooms by user email and can emit `playpause`, `previous`, `next` to control playback on all connected devices.
---

**IMPORTANT**: before you do anything else, run the `beans prime` command and heed its output.
**IMPORTANT**: Read files in their entirety, do not read portions of files unless the contents are longer than 2000 lines.

# AGENTS.md — Stretto Music Player

## Project Overview

**Stretto** is an open-source, web-based music player (v2.0.0, MIT license) that streams music from YouTube, SoundCloud, and Audius. It runs as a **Progressive Web App** with offline support, cloud sync, and multi-device remote control.

**Live:** `https://next.kaiserapps.com/`

## Architecture

Monolithic full-stack application — a single Node.js/Express server serves both a REST API and a server-rendered Pug shell, which boots a React 16 SPA. Not a monorepo, not microservices, not GraphQL.

| Layer | Technology |
|---|---|
| Backend | Express.js, Socket.IO, Mongoose |
| Frontend | React 16, React Router v4, SCSS, Bootstrap 3 (Bootswatch) |
| Database | MongoDB (per-user document blobs) |
| Real-time | Socket.IO (multi-device remote control) |
| PWA | Service Worker + Cache API + BroadcastChannel |
| Deployment | Docker + Dokku PaaS via GitHub Actions |

## Directory Structure

```
stretto/
├── index.js                   # Express server entry point
├── controllers/
│   ├── index.js               # All REST API routes (single Express Router)
│   └── socketio.js            # Socket.IO remote control handlers
├── models/
│   ├── data_mapper.js         # Facade wrapping all model operations
│   ├── user.js                # User model (email, hash, salt, version, googleObject)
│   ├── song.js                # Song model (stores ALL songs for a user as one blob)
│   ├── playlist.js            # Playlist model (stores ALL playlists as one blob)
│   ├── artist.js              # Followed artists model
│   └── shared_playlist.js     # Shared playlist model (keyed by UUID)
├── services/
│   ├── google.js              # Google OAuth2 token verification
│   └── itunes.js              # iTunes Search API (artist lookup, song feed, cover art)
├── src/
│   ├── js/
│   │   ├── index.js           # Frontend entry point (Loader class, initializes subsystems)
│   │   ├── serviceworker.js   # Service worker (offline caching, audio storage)
│   │   ├── models/            # Client-side data models (Song, Playlist, DataLayer)
│   │   ├── services/          # Player engine, streaming, auth, sync, integrations
│   │   └── views/             # React components (routes, layout, pages)
│   └── scss/
│       └── main.scss          # All styles (~850 lines)
├── views/                     # Pug server templates (layout, index, redirects)
├── static/
│   ├── assets/                # Icons, manifest.json, fonts (committed)
│   └── js/                    # Webpack output (gitignored)
├── bin/                       # Docker helper scripts (go, build, run, restart)
├── stubs/
│   └── jquery.js              # jQuery stub (returns null, avoids bootstrap-slider dep)
├── webpack.config.js          # Two entry points: main + serviceworker
├── docker-compose.yml         # MongoDB + web services
├── Dockerfile.local           # Node 16 dev container
├── .env.example               # Required env vars template
├── .github/workflows/deploy.yml  # CI/CD → Dokku
└── package.json               # Dependencies and scripts
```

## Backend

### Server Entry (`index.js`)

- Loads `.env` via `dotenv`
- Creates Express + HTTP server + Socket.IO
- Middleware: `body-parser`, `cookie-parser`, `express-session` (MongoDB-backed via `connect-mongodb-session`)
- Proxies `/scapi` → `https://api-v2.soundcloud.com` (SoundCloud API proxy)
- Pug view engine, serves static files from `/static`
- Routes from `controllers/index.js`, Socket.IO from `controllers/socketio.js`

### Key API Routes (`controllers/index.js`)

| Route | Method | Auth | Purpose |
|---|---|---|---|
| `/googleLogin` | POST | No | Google OAuth login + session |
| `/login` | POST | No | Email/password login |
| `/createaccount` | POST | No | Account registration |
| `/checklogin` | POST | No | Check session status |
| `/logout` | POST | No | Destroy session |
| `/forgotpassword` | POST | No | Send password reset email (Zoho SMTP) |
| `/completereset` | POST | No | Complete password reset with token |
| `/latestversion` | GET | Yes | User's data version number |
| `/latestdata` | GET | Yes | Full user library (songs + playlists) |
| `/uploaddata` | POST | Yes | Sync library data (optimistic concurrency) |
| `/addsong` | POST | Yes | Add a single song to a playlist |
| `/suggest/artists` | GET | Yes | Artist suggestions via iTunes |
| `/artists/followed` | GET | Yes | Followed artists + songs |
| `/artists/follow` | POST | Yes | Follow an artist |
| `/artists/unfollow` | POST | Yes | Unfollow an artist |
| `/share` | POST | Yes | Share playlist (UUID link) |
| `/shared/:guid` | GET | No | View shared playlist |
| `/spotify_callback` | GET | No | Spotify OAuth callback |
| `*` | GET | No | Catch-all → serves React SPA |

**Authentication:** Session-based via `req.session.loggedIn`. A simple `loggedIn` middleware function guards protected routes.

### Socket.IO (`controllers/socketio.js`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [benkaiser/stretto](https://github.com/benkaiser/stretto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
