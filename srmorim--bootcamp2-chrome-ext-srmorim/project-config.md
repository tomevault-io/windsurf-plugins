---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Bootcamp Pomodoro PWA** - Progressive Web App with Node.js backend implementing a Pomodoro timer with session history, statistics, and motivational quotes. Built as part of Bootcamp II final delivery.

**Version:** 2.0.0

**Stack:**
- Frontend: Vite + Vanilla JS + PWA Plugin
- Backend: Node.js + Express
- Testing: Playwright (E2E)
- Containerization: Docker + Docker Compose
- CI/CD: GitHub Actions + GitHub Pages

## Project Type

Progressive Web App (PWA) with RESTful API backend - Monorepo with workspaces

## Development Commands

### Setup

```bash
# Install all dependencies (root + apps)
npm install

# Or install individually
cd apps/web && npm install
cd apps/api && npm install
cd tests/e2e && npm install && npx playwright install chromium
```

### Development

```bash
# Run web dev server (Vite HMR on port 8080)
npm run dev:web

# Run API dev server (Node watch mode on port 3000)
npm run dev:api

# Run both in separate terminals for full development
```

### Build & Preview

```bash
# Build web PWA for production
npm run build:web

# Preview production build
cd apps/web && npm run preview
```

### Testing

```bash
# Run all E2E tests (requires web + API running)
npm run test:e2e

# Run API unit tests (16 tests - sessions + quotes)
npm run test:unit
# Or directly:
cd apps/api && npm test

# Run E2E tests with UI (interactive mode)
cd tests/e2e && npm run test:ui

# Run E2E tests in headed mode (see browser)
cd tests/e2e && npm run test:headed

# View Playwright report
cd tests/e2e && npm run report
```

### Lighthouse CI

```bash
# Run Lighthouse CI locally
npx @lhci/cli@0.12.x autorun --config=.lighthouserc.cjs

# Requires web server running on localhost:8080
```

### Docker

```bash
# Build and start all services
docker-compose up --build

# Or use npm scripts
npm run docker:up

# Stop services
npm run docker:down

# View logs
npm run docker:logs

# Access:
# - Web PWA: http://localhost:8080
# - API: http://localhost:3000
```

## Architecture

### Monorepo Structure

```
apps/
├── web/          # Frontend PWA (Vite)
│   ├── src/
│   │   ├── main.js        # UI initialization & event handlers
│   │   ├── timer.js       # Timer class (localStorage state)
│   │   ├── api-client.js  # API communication layer
│   │   └── styles.css
│   ├── public/icons/
│   ├── vite.config.js     # PWA manifest + service worker config
│   └── Dockerfile
│
├── api/          # Backend API (Express)
│   ├── src/
│   │   ├── index.js              # Express app & routes
│   │   ├── routes/sessions.js    # Sessions CRUD endpoints
│   │   └── services/quotes.js    # Quotable API proxy
│   └── Dockerfile
│
tests/e2e/        # E2E tests (Playwright)
├── pwa.spec.ts   # PWA functionality tests
└── api.spec.ts   # API endpoint tests

docker-compose.yml  # Multi-container orchestration
```

### Communication Flow

```
Browser PWA ←→ API Client ←→ Express API ←→ In-Memory Store
     ↓              ↓              ↓
localStorage   Service Worker  Quotable API
```

### State Management

**Frontend (apps/web/src/timer.js):**
- Timer state persisted to `localStorage` (mode, timeRemaining, isRunning, sessionsCompleted)
- Settings persisted to `localStorage` (durations, sound, notifications)
- `Timer` class manages state + tick interval + callbacks

**Backend (apps/api/src/routes/sessions.js):**
- Sessions stored in-memory array (resets on server restart)
- Simple CRUD operations with validation
- Statistics aggregation on-demand

### PWA Configuration

**Service Worker (apps/web/vite.config.js):**
- Workbox with `autoUpdate` strategy
- Caches: static assets, API responses (NetworkFirst), quotes API
- Offline support with cached fallbacks

**Manifest:**
- `base: '/bootcamp2-chrome-ext-SrMorim/'` for GitHub Pages deployment
- Standalone display mode
- Icons: 192x192 and 512x512 (maskable)

## API Endpoints

**Base URL:** `http://localhost:3000`

- `GET /api/health` - Health check
- `GET /api/quote` - Get random motivational quote (proxied from Quotable)
- `GET /api/sessions` - List all sessions (sorted by completedAt desc)
- `GET /api/sessions/:id` - Get specific session
- `GET /api/sessions/stats` - Get statistics (total, today, thisWeek, byMode)
- `POST /api/sessions` - Create session (body: `{mode, duration, completedAt}`)
- `DELETE /api/sessions/:id` - Delete session
- `DELETE /api/sessions` - Clear all sessions

**Validation:**
- `mode` must be: `focus`, `shortBreak`, or `longBreak`
- `duration` must be positive number
- Returns 400 for validation errors, 404 for not found, 500 for server errors

## Important Files & Patterns

### Frontend Entry Point (apps/web/src/main.js)

Main UI controller that:
- Initializes `Timer` instance
- Handles button clicks (start, pause, reset, skip, mode switching)
- Updates DOM every second (timer display, mode UI, session counter)
- Fetches and displays motivational quotes
- Manages session history view and statistics
- Calls `apiClient` to persist completed sessions

### Timer Logic (apps/web/src/timer.js)

Timer class with:
- State persistence via localStorage
- Tick callbacks (called every second when running)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SrMorim) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
