---
trigger: always_on
description: AGNT is an Electron-based desktop AI agent framework for building, deploying, and orchestrating intelligent workflows. It combines a Vue.js frontend, Express.js backend, and Electron shell to provide a cross-platform desktop application for managing AI agents, workflows, and plugins.
---

# Claude AI Instructions for AGNT

## Project Overview

AGNT is an Electron-based desktop AI agent framework for building, deploying, and orchestrating intelligent workflows. It combines a Vue.js frontend, Express.js backend, and Electron shell to provide a cross-platform desktop application for managing AI agents, workflows, and plugins.

**Version**: 0.5.0
**Platform**: Windows, macOS, GNU/Linux
**Website**: https://agnt.gg
**Author**: Nathan Wilbanks

### Local-First Architecture

**AGNT is designed for:**

- ✅ Single users - Personal desktop or laptop
- ✅ Families - Shared Docker backend across household
- ✅ Small teams - 2-10 people in same organization

**NOT designed for:**

- ❌ Multi-tenant SaaS - Hundreds of unrelated users
- ❌ Public hosting - Each org self-hosts their own
- ❌ Large enterprises - 50+ concurrent users

**Why?** Uses SQLite (local database) and broadcasts real-time updates to all connected clients. Perfect for trusted groups sharing a workspace, not for isolating thousands of separate users.

## Architecture

### Tech Stack

- **Desktop Shell**: Electron (v33.0.2)
- **Backend**: Express.js + Node.js (ES Modules)
- **Frontend**: Vue.js 3 + Vite
- **State Management**: Vuex
- **Database**: SQLite3 (local storage)
- **Testing**: Playwright (E2E)

### Application Structure

```
┌─────────────────────────────────────┐
│   Electron Main Process (main.js)   │
│   - Window management                │
│   - IPC handlers                     │
│   - Auto-update system               │
└─────────────────────────────────────┘
            ↕ IPC (preload.js)
┌─────────────────────────────────────┐
│   Backend Server (Express on 3333)  │
│   - REST API routes                  │
│   - Plugin system                    │
│   - Workflow engine                  │
│   - AI provider integrations         │
└─────────────────────────────────────┘
            ↕ HTTP/WebSocket
┌─────────────────────────────────────┐
│   Frontend (Vue.js)                  │
│   - Agent UI                         │
│   - Workflow designer                │
│   - Plugin marketplace               │
└─────────────────────────────────────┘
```

## Development Workflow

### Quick Start (Development Mode)

```bash
# Terminal 1: Start frontend dev server (hot reload)
cd frontend
npm run dev          # Runs on http://localhost:5173

# Terminal 2: Start Electron app (loads dev server)
npm start            # Backend runs on port 3333
```

**Frontend Dev Server** (recommended for rapid iteration):

- Frontend runs on Vite dev server (port 5173)
- Hot module replacement (HMR) for instant updates
- Electron window loads from dev server

**Production Mode** (test built frontend):

```bash
# Build frontend first
cd frontend && npm run build && cd ..

# Start Electron (loads from frontend/dist)
npm start
```

### User Manages Dev Server

- **DO NOT** start/stop the frontend dev server automatically
- User controls `npm run dev` in a separate terminal
- Frontend changes appear instantly via HMR
- If frontend not loading, ask user to check their dev server

## Project Structure

```
/
├── main.js                    # Electron main process entry
├── preload.js                 # IPC bridge for renderer
├── package.json               # Root project config
├── .env.example               # Environment template
│
├── backend/                   # Express.js backend server
│   ├── server.js              # Server entry (port 3333)
│   ├── src/
│   │   ├── routes/            # API route handlers
│   │   ├── services/          # Business logic
│   │   ├── models/            # Data models
│   │   ├── plugins/           # Plugin management
│   │   ├── tools/             # Built-in workflow tools
│   │   ├── workflow/          # Workflow engine
│   │   ├── stream/            # WebSocket streaming
│   │   └── utils/             # Utilities
│   └── plugins/
│       ├── dev/               # Plugin development
│       ├── plugin-builds/     # Built .agnt packages
│       └── installed/         # User-installed plugins
│
├── frontend/                  # Vue.js frontend app
│   ├── src/
│   │   ├── views/             # Page components
│   │   ├── components/        # Reusable components
│   │   ├── store/             # Vuex state management
│   │   ├── services/          # API client services
│   │   └── router/            # Vue Router config
│   ├── dist/                  # Build output (served by Electron)
│   └── package.json           # Frontend dependencies
│
├── build/                     # Electron builder resources
│   ├── icon.ico               # Windows icon
│   ├── icon.icns              # macOS icon
│   ├── icons/                 # GNU/Linux icons
│   └── installer.nsh          # NSIS installer config
│
├── tests/                     # Playwright E2E tests
├── docs/                      # Documentation
└── scripts/                   # Build scripts
```

## Common Commands

### Development

```bash
npm start                      # Start Electron app (dev or prod mode)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agnt-gg/agnt](https://github.com/agnt-gg/agnt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
