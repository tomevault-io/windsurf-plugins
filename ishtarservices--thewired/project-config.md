---
trigger: always_on
description: The Wired V1 is a decentralized Nostr-native media platform. The repo is a **pnpm monorepo** with:
---

# CLAUDE.md -- Project Instructions for Claude Code

## Project Overview

The Wired V1 is a decentralized Nostr-native media platform. The repo is a **pnpm monorepo** with:

| Service | Language | Port | Directory |
|---------|----------|------|-----------|
| Client | TypeScript/React/Tauri | 1420 | `client/` |
| Relay | Rust (axum + tokio + sqlx) | 7777 | `services/relay/` |
| Backend | Node.js/TypeScript (Fastify + Drizzle) | 3002 | `services/backend/` |
| Gateway | Go (NIP-98 auth + rate limiting) | 9080 | `services/gateway/` |
| Shared Types | TypeScript | - | `packages/shared-types/` |
| Landing | Astro + React | - | `services/landing/` |
| Proxy | Caddy (prod) | - | `services/proxy/` |

Infrastructure: PostgreSQL (5432), Redis (6380), Meilisearch (7700), LiveKit (7880) via `docker-compose.yml`.

The Rust relay also compiles to an **embedded SQLite** build (Cargo `embedded` feature) that runs in-process inside the Tauri client (port 7787) for self-hosted spaces — see `client/src-tauri/src/relay.rs`.

## Build & Development

```bash
pnpm install              # Install all workspace dependencies (from root)
pnpm dev                  # PRIMARY: full backend stack via process-compose TUI
                          #   (infra + native LiveKit + relay + backend + gateway, health-gated)
pnpm dev:client           # Vite dev server (web only, port 1420) — run in its own terminal
pnpm dev:backend          # Backend service with tsx watch
pnpm dev:gateway          # Go gateway
pnpm dev:relay            # Rust relay
pnpm dev:infra            # Start Postgres + Redis + Meilisearch + LiveKit (Docker, detached)
pnpm dev:services         # Start all Docker services
pnpm dev:down             # Stop the process-compose app processes (infra stays up)
pnpm build                # Build all packages
pnpm typecheck            # Typecheck all TypeScript packages
```

`pnpm dev` is the recommended path (one scrollable log pane per process, dependency ordering). The
client runs separately (`pnpm dev:client` or `cd client && pnpm tauri dev`). See README §Setup for the
process-compose TUI cheatsheet.

### Client (Tauri)

```bash
cd client && pnpm tauri dev    # Full Tauri desktop app with hot reload
cd client && pnpm tauri build  # Production desktop app bundle
```

### Type Checking

```bash
pnpm typecheck                           # All packages
pnpm --filter @thewired/client typecheck # Client only
pnpm --filter @thewired/backend typecheck # Backend only
```

### Rust (Relay)

```bash
cd services/relay && cargo check   # Check Rust compilation
cd services/relay && cargo build   # Build Rust binary
```

### Rust (Client Tauri)

```bash
cd client/src-tauri && cargo check   # Check Rust compilation
cd client/src-tauri && cargo build   # Build Rust binary
```

**Warning**: `cargo` commands change cwd. Use absolute paths for subsequent commands.

## Monorepo Structure

```
/
├── client/                    # Tauri + React client app
│   ├── src/                   # React source
│   ├── src-tauri/             # Rust Tauri backend
│   ├── package.json           # @thewired/client
│   ├── tsconfig.json          # Extends ../tsconfig.base.json
│   └── vite.config.ts
├── packages/
│   └── shared-types/          # @thewired/shared-types (type-only package)
│       └── src/               # nostr.ts, space.ts, profile.ts, api.ts, permissions.ts, music.ts
├── services/
│   ├── backend/               # @thewired/backend (Fastify + Drizzle + PostgreSQL)
│   │   └── src/               # routes/, services/, workers/, db/, middleware/, lib/
│   ├── gateway/               # Go API gateway (NIP-98 + rate limiting)
│   │   └── internal/          # auth/, ratelimit/, proxy/, cors/, logging/
│   ├── relay/                 # Rust NIP-29 relay (axum + sqlx; pg + embedded sqlite)
│   │   └── src/               # nostr/, protocol/, db/, music/
│   ├── landing/               # Astro marketing site
│   └── proxy/                 # Caddy production proxy config
├── config/livekit.yaml        # LiveKit SFU config
├── docs/                      # Design notes, plans, roadmaps (AI_ENGINE, DECENTRALIZED_SPACES, …)
├── docker-compose.yml
├── pnpm-workspace.yaml
├── tsconfig.base.json
├── package.json               # Root workspace scripts
├── CLAUDE.md
└── ARCHITECTURE.md
```

## Client Structure (`client/src/`)

- `app/` -- App root, layout, routing
- `components/layout/` -- Shell components (Sidebar, CenterPanel, RightPanel, TopBar)
- `components/ui/` -- Shared primitives (Button, Avatar, Spinner)
- `features/` -- Feature modules, each self-contained (key ones; see the tree in ARCHITECTURE.md for the full list):
  - `chat/` -- Kind:9 real-time chat with optimistic UI
  - `identity/` -- Login, multi-account switcher, signer detection
  - `dm/` -- NIP-17 encrypted DMs, contacts, friend requests
  - `spaces/` -- NIP-29 spaces, channels, members, moderation; **three space modes** (see below)
  - `music/` -- Music library, player, upload (kinds 31683/33123/30119)
  - `voice/` + `calling/` -- LiveKit voice/video channels + 1:1 DM WebRTC calls
  - `ai/` -- Toggleable AI assistant: engine, providers, gated tools, artifacts (see below)
  - `wallet/` -- NIP-47 NWC wallet + NIP-57 zaps

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ishtarservices/TheWired](https://github.com/ishtarservices/TheWired) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
