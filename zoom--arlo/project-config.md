---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Arlo Meeting Assistant** is an open-source Zoom Apps reference implementation demonstrating how to build intelligent meeting assistants that capture real-time transcripts using RTMS (Real-Time Media Streams) — **without requiring a meeting bot**. The app runs natively inside Zoom meetings and provides AI-powered summaries, action items, and transcript search.

**Current Phase:** v1.0 — see [`SPEC.md`](./SPEC.md) for the full feature specification and version milestones.

## Development Commands

### Docker Setup (Recommended)

```bash
docker-compose up --build              # Start all services (Postgres, Backend, Frontend, RTMS)
docker-compose up --build -V           # Rebuild with fresh node_modules (use after adding/removing npm deps)
docker-compose logs -f backend         # View backend logs
docker-compose restart backend         # Restart specific service
docker-compose down -v && docker-compose up --build  # Clean restart (deletes DB data)
```

### Manual Development

```bash
npm run dev              # Start all services concurrently (backend + frontend + rtms)
npm run dev:backend      # Backend only (nodemon for auto-restart)
npm run dev:frontend     # Frontend only (CRA dev server)
npm run dev:rtms         # RTMS service only
npm run setup            # Install all workspace dependencies
```

### Database (Prisma)

```bash
npm run db:migrate       # Run migrations (from root)
npm run db:generate      # Generate Prisma client after schema changes
npm run db:studio        # Open Prisma Studio GUI at localhost:5555
npm run db:reset         # Reset database (WARNING: deletes all data)

# Or from backend directory:
cd backend
npx prisma migrate dev --name description_of_change   # Create new migration
```

### Frontend Build

```bash
cd frontend && npx react-scripts build
```

### ngrok (Required for Zoom App Testing)

```bash
ngrok http 3000                                        # Random domain (changes each restart)
ngrok http 3000 --domain=yourname-arlo.ngrok-free.app  # Static domain (recommended)
# Then update PUBLIC_URL in .env
```

## Architecture

### System Components

1. **In-Meeting Zoom App** (`frontend/`) — React 18 + Base UI + Zoom Apps SDK
   - Runs embedded in Zoom client during meetings
   - Live transcript display, AI suggestions, highlights
   - Start/stop RTMS via `zoomSdk.callZoomApi('startRTMS')`

2. **Backend API** (`backend/`) — Node.js/Express + PostgreSQL + Prisma
   - Zoom OAuth 2.0 (PKCE flow), session management with httpOnly cookies
   - REST API for meetings, transcripts, search, AI, highlights
   - WebSocket server for live transcript broadcast
   - AI orchestration via OpenRouter (free models, no API key required)

3. **RTMS Service** (`rtms/`) — @zoom/rtms v1.0.2
   - Webhook handlers for `meeting.rtms_started` / `meeting.rtms_stopped`
   - WebSocket-based transcript ingestion from Zoom

**Note:** A Post-Meeting Web App (Next.js) is planned but not yet implemented.

### Data Flow

```
Zoom RTMS WebSocket → RTMS Service → Backend (normalize, buffer, batch insert to Postgres)
    → WebSocket broadcast → Frontend (live transcript display, < 1s end-to-end)
```

### Authentication Flow (Zoom OAuth PKCE)

Implemented in `useZoomAuth` hook (`frontend/src/hooks/useZoomAuth.js`) — single source of truth for auth.

```
1. Frontend: GET /api/auth/authorize → { codeChallenge, state }
2. Frontend: Register onAuthorized listener BEFORE calling authorize() (avoids race condition)
3. Frontend: zoomSdk.authorize({ codeChallenge, state })
4. Zoom fires onAuthorized → { code } (NOTE: SDK does NOT return state — use closure from step 1)
5. Frontend: POST /api/auth/callback { code, state } (credentials: 'include')
6. Backend: Exchanges code for tokens, stores AES-256-GCM encrypted in Postgres, creates session cookie
7. Frontend: login(user, wsToken) → navigate to /home
```

**Session restoration:** On app load, `AuthContext` calls `GET /api/auth/me` to restore session from httpOnly cookie. A loading spinner displays during this check to prevent auth-screen flash.

**User info fallback:** If `user:read` OAuth scope is not configured, backend decodes JWT access token payload for user ID and name.

## Key Files & Architecture Details

### Backend (`backend/src/`)
- `server.js` — Express app setup, middleware, route mounting, rate limiting, graceful shutdown
- `config.js` — Environment variable validation
- `lib/prisma.js` — Singleton PrismaClient (all route/service modules import from here)
- `routes/` — 9 route modules: auth, meetings, ai, home, rtms, search, highlights, zoom-meetings, preferences
- `services/` — auth (token/PKCE/encryption), openrouter (LLM), websocket (broadcast), zoomApi (Zoom REST helper with token refresh + mutex)
- `middleware/auth.js` — `requireAuth` and `optionalAuth` session middleware

### Frontend (`frontend/src/`)
- `App.js` — HashRouter, route definitions, provider hierarchy: Theme → ZoomSdk → Auth → Meeting → Toast
- `index.css` — Design tokens, typography (Source Serif 4 + Inter), light/dark theme variables

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zoom/arlo](https://github.com/zoom/arlo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
