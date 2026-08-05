---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Repository Overview

Android remote device management platform with three components:
- **`server/`** — Go backend (Gin + GORM + SQLite/MySQL)
- **`web/`** — Vue 3 frontend (Vite + Element Plus + Pinia)
- **`agent/`** — Android Kotlin agent app (OkHttp WebSocket + MediaProjection)

## Build Commands

### Server (Go 1.21+)
```bash
make server              # build web first, then go build → bin/app-manager
make server-only         # go build without rebuilding web
make test                # go test ./...
make fmt                 # go fmt ./...
make check               # fmt + test + go vet
# Run directly:
cd server && go run . ../server/config.sqlite.yaml
```

Cross-compile targets: `make server-linux-amd64`, `make server-linux-arm64`, `make server-darwin-amd64`, `make server-darwin-arm64`

### Web (Vue 3 / Vite)
```bash
cd web && npm install
npm run dev              # dev server, proxies /api and /ws to http://127.0.0.1:8080
npm run build            # production build → web/dist/
```

Proxy target can be overridden with `VITE_PROXY_TARGET` env var.

### Web — SCADA (组态)

Standalone React app in `scada-editor/` (Vite + React + Zustand + TanStack Query). Served at `/scada-editor/` by the Go server; opened from the Vue shell via `openScadaEditor()` in `Layout.vue`.

```bash
cd scada-editor && npm install
npm run dev      # dev server (proxies /api to http://127.0.0.1:8080)
npm run build    # production build → scada-editor/dist/
```

- **Pages**: `ScadaListPage` (`/scada`), `EditorPage` (`/editor/:id`), `PreviewPage` (`/preview/:id`), `SchemaPage` (`/schema`).
- **Store**: `src/store/editorStore.ts` (Zustand) — multi-canvas project, undo/redo history, element CRUD, z-order, lock/visibility.
- **Canvas**: `CanvasBoard.tsx` — HTML5 Canvas 2D rendering, drag/resize/marquee select, right-click context menu (z-order + delete), locked element guard.
- **Widgets**: `WidgetPanel.tsx` (drag-to-canvas), `ChartWidget.tsx` (ECharts), `ImageWidget.tsx` (image-bg/widget/decoration/border-box).
- **Sim engine**: `server/scada/sim.go` — `StartSimEngine()` started after `database.Ready`; pushes point data via STOMP `/topic/scada/point-data/:code`.
- **REST**: `/api/scada/*` (Gin routes in `server/api/scada.go`).

### Agent (Android)
```bash
make agent               # assembleDebug → agent/app/build/outputs/apk/debug/
make agent-release       # assembleRelease
make install-agent       # installDebug via ADB
```

### Form App (React)
```bash
cd form-app && npm install
npm run dev              # dev server at :5175, proxies /api to http://127.0.0.1:8080
npm run build            # production build → form-app/dist/
```

## Development Environment

Entry point: **http://localhost:3001** (web/Vue 3)

| Port | Service | Role |
|------|---------|------|
| `:3001` | web (Vite) | Browser entry |
| `:5175` | form-app (Vite) | Form designer dev server |
| `:8080` | server (Go) | Backend API |

### Bridge (USB Scanner)
```bash
cd bridge && go build -o app-manager-bridge .
./app-manager-bridge     # listens on ws://127.0.0.1:17175, local-only loopback
```
Bridge discovers USB-connected Android devices via ADB and pushes them to the browser for one-click registration. Only listens on `127.0.0.1`.

### Release Packaging
```bash
make release             # web + server + agent → dist/release/app-manager-<VERSION>/
make release-zip         # + zip archive
make release-tar         # + tar.gz archive
make clean
```

## Configuration

Server reads a YAML config file passed as the first CLI argument. SQLite quickstart: `server/config.sqlite.yaml`. Key fields:

```yaml
server:
  port: 8080
  host: 0.0.0.0
database:
  type: sqlite          # or mysql
  dsn: ./data/app-manager.db
storage:
  path: ./uploads
adb:
  path: adb
ffmpeg:
  path: ""              # optional, for server-side recording
jwt:
  secret: change-me-in-production
```

Env overrides: `JWT_SECRET`, `ADB_PATH`, `FFMPEG_PATH`.

Default admin: `admin / admin123` (auto-created on first run).

### MQTT (optional)

Custom events can forward to MQTT broker:

```yaml
mqtt:
  enabled: true
  broker: tcp://localhost:1883
  username: ""
  password: ""
  client_id: app-manager
  qos: 1
```

Event groups and individual events can define MQTT topics in the UI; events are forwarded on publish.

## Architecture

### Component Interaction

```
Browser (Vue 3)
  │  REST /api/*
  │  WS /ws/stomp          — STOMP push notifications
  │  WS /ws/screen/:id     — MJPEG frame stream
  │  WS /ws/shell/:id      — PTY shell (xterm.js)
  │  WS /ws/logcat/:id     — logcat stream
  ▼
Go Server (Gin)
  │  GORM (SQLite or MySQL, AutoMigrate on startup)
  │  ADB subprocess client
  │  ffmpeg subprocess (optional, server-side recording)
  │  task.Queue — 5 goroutine workers, buffered chan uint(100)
  │  AgentHub — per-device WebSocket connection map
  │  ScreenHub — per-device viewer fan-out
  ▼
Android Agent (OkHttp WebSocket, persistent connection)
  WS /ws/agent/:deviceToken
```

### Device Modes

- **ADB-only**: device registered with ADB serial; server shells/installs via `adb` subprocess.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qianwensoft/app_manager](https://github.com/qianwensoft/app_manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-04 -->
