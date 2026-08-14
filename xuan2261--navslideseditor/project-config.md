---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working on code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working on code in this repository.

## Project Overview

**NavSlides Editor** — a self-hostable WYSIWYG presentation editor powered by reveal.js. Built as an npm workspace monorepo with 4 packages: `client` (React/Vite), `server` (Express), `shared` (pure Node.js utilities), and `electron` (desktop shell).

## Commands

### Development

```bash
npm run dev          # Start Vite dev server (5173) + Express API (3002) concurrently
npm run build        # Compile React → client/dist/
npm start            # Serve built client + API on port 3002 (production)
```

### Testing

```bash
npm run test                   # Unit tests (Vitest)
npm run test:e2e               # E2E tests (Playwright)
npm run test:load:api          # k6 load test: REST API
npm run test:load:ws           # k6 load test: WebSocket/Socket.IO
npm run test:corpus            # PPTX import semantic & roundtrip fidelity test
```

### Linting & Formatting

```bash
npm run lint     # ESLint
npm run format   # Prettier
```

### Electron Desktop App

```bash
npm run electron:dev        # Run Electron in dev mode (no package)
npm run electron:build:win   # Build Windows .exe installer
npm run electron:build:linux # Build Linux .AppImage + .deb
npm run electron:build:mac   # Build macOS .zip
```

### Docker

```bash
docker compose up -d          # Start server on port 3002 with persistent volumes
docker compose logs -f         # Tail logs
docker compose down -v        # Stop + delete volumes
```

### Single Test

```bash
npx vitest run server/routes/share.test.js
npx vitest run shared/tests/
npx playwright test tests/smoke.spec.js
```

## Architecture

### Monorepo Structure

```
NavSlidesEditor/
├── client/           # React SPA (Vite). Exports to client/dist/ for production
│   └── src/
│       ├── pages/           # EditorPage, HomePage, LiveViewPage, SpeakerViewPage, etc.
│       ├── components/      # SlideCanvas, PropertiesPanel, SlidePanel, ribbon/, etc.
│       ├── stores/          # editor-store, presentation-store, ui-store (Zustand)
│       ├── hooks/           # use-keyboard, use-clipboard, use-live-presentation, etc.
│       └── extensions/      # TipTap: FontSize, FontFamily, MathExtension, etc.
├── server/           # Express REST API + WebSocket (Socket.IO)
│   ├── index.js            # Main server (323 LOC) — imports modular routes
│   ├── routes/             # presentations, templates, share, upload, github, sync, 
│   │                       # history, settings, media, live, pptx-import, games, ai, etc.
│   ├── services/           # storage, socket-handler, live-rooms, pptx-exporter, etc.
│   └── middleware/         # error-handler, etc.
├── shared/           # Pure Node.js utilities shared between client & server
│   └── src/
│       ├── htmlGenerator.js      # JSON presentation → reveal.js HTML
│       ├── element-renderers.js  # Element-specific HTML rendering
│       ├── shapeUtils.js         # SVG shape generation
│       ├── presenterTools.js     # Presenter mode utilities
│       ├── content-safety.js     # Content validation
│       └── shared-*.js           # PPTX, color, text utilities
└── electron/         # Desktop Electron shell, embeds the server
```

`shared` is consumed by both `client` (during Vite build/export) and `server` (at runtime) via npm workspace symlinks.

### Client Architecture (React)

- **Routing** (`App.jsx`): `react-router-dom` v7. Routes: `/` (Home), `/editor/:id`, `/template/:id`, `/settings`, `/explore`, `/live/:roomCode`, `/remote/:roomCode`, `/speaker/:roomCode`, `/game/join`.
- **State**: Zustand stores in `client/src/stores/`:
  - `editor-store.js` — selection, clipboard, grid/guides, timeline, find-replace
  - `presentation-store.js` — presentation data (loaded/saved via REST)
  - `ui-store.js` — UI state (includes ribbon state)
- **Pages** (`client/src/pages/`):
  - `EditorPage.jsx` — main editor (77k LOC; large file)
  - `HomePage.jsx` — dashboard, CRUD, templates (68k LOC)
  - `LiveViewPage.jsx`, `RemoteControlPage.jsx`, `SpeakerViewPage.jsx` — live presentation
  - `SettingsPage.jsx`, `ExplorePage.jsx`, `game-player-join-page.jsx`
- **Components** (`client/src/components/`): SlideCanvas, PropertiesPanel, SlidePanel, QuickAccessToolbar, AnimationTimeline, FindReplaceBar, ShareModal, ribbon/ (new UI), various modals (AI, media, templates, etc.).
- **Hooks** (`client/src/hooks/`): use-keyboard, use-clipboard, use-slide-operations, use-live-presentation, use-game-socket, use-annotation-sync, etc.
- **Extensions** (`client/src/extensions/`): TipTap extensions — FontSize, FontFamily, MathExtension (KaTeX), font-weight, line-height.
- **Vite proxy**: `/api`, `/uploads`, `/vendor`, `/ws` → `localhost:3002` in dev.

### Server Architecture (Express)

Main `server/index.js` (323 LOC) imports modular routes from `server/routes/`:

- **Routes** (`server/routes/`):
  - `presentations.js` — GET/POST/PUT/DELETE /api/presentations
  - `templates.js` — GET/POST/DELETE /api/templates
  - `share.js` — POST /api/share/:id (generate/revoke share tokens)
  - `upload.js` — POST /api/upload (file upload via multer, SHA256 deduplication)
  - `github.js` — POST /api/github/push

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xuan2261/NavSlidesEditor](https://github.com/xuan2261/NavSlidesEditor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
