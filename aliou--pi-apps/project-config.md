---
trigger: always_on
description: Clients for the [pi](https://github.com/mariozechner/pi-coding-agent) coding agent.
---

# Pi Apps

Clients for the [pi](https://github.com/mariozechner/pi-coding-agent) coding agent.

**Scope:** Single-user personal deployment. No multi-user, no user accounts, no per-user isolation.

## Protocol Boundaries (READ FIRST)

This project has **two distinct communication layers**. Understanding when to modify each is critical.

### RPC Protocol (NEVER MODIFY)

The RPC protocol is defined by the upstream [pi-coding-agent](https://github.com/mariozechner/pi-coding-agent) package.

- **Source of truth:** `@anthropic/pi-coding-agent` npm package (see `pi-mono` repo)
- **Rule:** NEVER add, remove, or modify RPC message types. If upstream changes, update any mirrors to match.
- **Examples:** `AgentEvent`, `ClientCommand`, `ToolCall`, `Message`, etc.

The WebSocket connection (`ws://host/ws/sessions/:id`) is a **transparent proxy** for RPC messages. It does not define a new protocol - it just transports the same JSON-RPC messages that pi uses over stdin/stdout.

### REST API (CAN EXTEND)

The REST API (`/api/*`) is our custom relay server infrastructure. This is where we add features like environments, sessions, repos, secrets.

- **Location:** `server/relay/src/routes/`
- **Rule:** Add new endpoints freely, but keep REST concerns separate from RPC concerns.
- **Examples:** `/api/sessions`, `/api/environments`, `/api/github/repos`

### Quick Reference

| Layer | Can Modify? | Where Defined | Purpose |
|-------|-------------|---------------|---------|
| RPC Protocol | NO | pi-coding-agent upstream | Agent communication |
| WebSocket Transport | NO | Proxy only | Carries RPC over network |
| REST API | YES | relay-server | Session/resource management |

See `server/relay/AGENTS.md` for package-specific guidance.

## Architecture

- **Relay** wraps Pi sessions, manages repos (cloned from GitHub), and exposes REST API + WebSocket protocol for remote clients.
- **Dashboard** is the web admin UI for managing secrets, GitHub token, and viewing sessions.
- **Native** apps (iOS/macOS) are built with Swift/SwiftUI, using XcodeGen for project generation.
- **Gondolin** is a microVM sandbox provider using `@earendil-works/gondolin` for lightweight ARM64 VMs (no Docker dependency).

## Xcode Workspace

`clients/native/PiApps.xcworkspace` ties together the native app and Swift packages. When adding a new Swift package to `clients/native/packages/` or a new native app, add a `<FileRef>` entry to `clients/native/PiApps.xcworkspace/contents.xcworkspacedata`.

## Build

TypeScript apps are independent with their own dependencies and build commands. Use `just dev` from the repo root to start both the relay server and dashboard in parallel with hot reload.

**Relay Server:**
```bash
cd server/relay
pnpm install       # install dependencies
pnpm dev           # run dev server (hot reload)
pnpm build         # build
pnpm lint          # lint (biome)
pnpm typecheck     # typecheck (tsc)
pnpm test          # test (vitest)
```

**Dashboard:**
```bash
cd clients/dashboard
pnpm install       # install dependencies
pnpm dev           # run dev server (hot reload)
pnpm build         # build
pnpm typecheck     # typecheck (react-router + tsc)
```

**Native (iOS/macOS):**
```bash
just setup         # generate Xcode project
just build         # build macOS (debug)
just build-ios     # build iOS simulator (debug)
just test          # run native tests
```

## Docker Images

Published to GitHub Container Registry on pushes to `main`. Each has a workflow in `.github/workflows/` and a Dockerfile in its source directory.

| Image | Source | Port |
|-------|--------|------|
| `ghcr.io/aliou/pi-relay` | `server/relay/` | 31415 |
| `ghcr.io/aliou/pi-dashboard` | `clients/dashboard/` | 8080 |
| `ghcr.io/aliou/pi-sandbox-alpine-arm64` | `server/sandboxes/docker/sandbox-alpine-arm64/` | - |
| `ghcr.io/aliou/pi-sandbox-codex-universal` | `server/sandboxes/docker/sandbox-codex-universal/` | - |
| `ghcr.io/aliou/pi-sandbox-cloudflare` | `server/sandboxes/cloudflare/` | - |
| `ghcr.io/aliou/pi-sandbox-gondolin` | `server/sandboxes/gondolin/` | - |

**CI Jobs:** Gondolin assets are built and probed in the `relay-gondolin-assets` job in `ci.yml`, not via a separate workflow.

## Structure

- `server/relay/` - Relay API server (Node.js/Hono/SQLite)
- `server/sandboxes/cloudflare/` - Cloudflare Containers sandbox (Worker, bridge, Dockerfile)
- `server/sandboxes/docker/` - Docker sandbox images for local/self-hosted relay
- `server/sandboxes/gondolin/` - Gondolin microVM sandbox assets and build config
- `server/scripts/` - Utility scripts (nuke-sessions, list-containers)
- `clients/dashboard/` - Relay admin UI (React Router v7 SPA/Vite/Tailwind)
- `clients/native/` - Native Swift apps and packages (iOS, macOS)

## Relay API

The relay server exposes REST endpoints and WebSocket for session communication.

**REST Endpoints:**
- `GET /health` - Server health check
- `GET /api/sessions` - List sessions
- `POST /api/sessions` - Create session
- `GET /api/sessions/:id` - Get session
- `POST /api/sessions/:id/activate` - Activate session (ensure sandbox running, blocks until ready)
- `POST /api/sessions/:id/archive` - Archive session (stop sandbox, mark as archived)
- `DELETE /api/sessions/:id` - Hard delete session from DB

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aliou/pi-apps](https://github.com/aliou/pi-apps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
