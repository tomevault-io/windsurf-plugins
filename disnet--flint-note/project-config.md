---
trigger: always_on
description: This is a **unified Electron application** containing a Svelte-based UI and the integrated Flint note server. The project uses a single package structure for simplified development and building.
---

# Flint UI Project

## Project Overview

This is a **unified Electron application** containing a Svelte-based UI and the integrated Flint note server. The project uses a single package structure for simplified development and building.

## Automerge-Based Architecture

The app uses Automerge for local-first data storage with CRDT-based data structures:

- **Entry point**: `src/renderer/src/App.svelte`
- **Data storage**: Automerge with IndexedDB (`src/renderer/src/lib/automerge/`)
- **State management**: Unified state module in `state.svelte.ts`

See `docs/AUTOMERGE-MIGRATION.md` for architecture details and history.

## Project Structure

```
flint-ui/
├── package.json                 # Single package configuration
├── src/
│   ├── main/                   # Electron main process
│   ├── preload/                # Electron preload scripts
│   ├── renderer/               # Svelte UI application
│   │   ├── index.html         # Main HTML file
│   │   └── src/               # Svelte source code
│   │       ├── components/    # Svelte components
│   │       ├── services/      # API and service layers
│   │       ├── stores/        # Svelte stores
│   │       └── utils/         # Utility functions
│   └── server/                 # Integrated note server
│       ├── api/               # Server API layer
│       ├── core/              # Core note logic
│       ├── database/          # Database management
│       ├── server/            # Server handlers
│       ├── types/             # Type definitions
│       └── utils/             # Server utilities
├── sync-server/                # Separate Bun project for cloud sync
│   ├── src/
│   │   ├── index.ts           # Express server entry point
│   │   ├── db.ts              # SQLite database (bun:sqlite)
│   │   ├── auth/              # Bluesky ATProto OAuth, sessions, invite codes
│   │   └── sync/              # Sync implementation
│   │       ├── lean-sync-server.ts  # WebSocket sync (one-doc-at-a-time)
│   │       ├── doc-storage.ts       # Document binary storage
│   │       ├── file-storage.ts      # Binary file storage (PDFs, images, etc.)
│   │       ├── file-routes.ts       # REST API for file upload/download
│   │       ├── vault-access.ts      # Document access control
│   │       ├── document-registration.ts # Document registration API
│   │       └── diagnostics.ts       # Server diagnostics endpoints
│   └── tests/
├── docs/                       # Project documentation
└── [config files]             # Build configs, TypeScript, etc.
```

## Development Commands

- `npm run build` - Build the complete application
- `npm run dev` - Start development server
- `npm run lint` - Run linter on all source code
- `npm run typecheck` - Run TypeScript checking
- `npm run format` - Format code across all files
- `npm run clean` - Clean build artifacts
- `npm run test` - Run tests in watch mode
- `npm run test:run` - Run tests once

## System Layout

### Documentation

- `docs/FLINT-OVERVIEW.md` - Design philosophy and core concepts
- `docs/ARCHITECTURE.md` - Electron system architecture documentation
- `docs/FLINT-NOTE-API.md` - Server API documentation
- `docs/LEAN-SYNC-SERVER.md` - Lean sync server architecture and wire protocol

### Source Code

- `src/main/` - Electron main process with AI and note services
- `src/preload/` - Preload scripts for secure IPC
- `src/renderer/` - Svelte UI application
- `src/server/` - Integrated note server with API, database, and core logic

### Sync Server

- `sync-server/` - **Separate Bun project** (not part of the Electron build)
- Runtime: Bun, deployed to Fly.io
- Auth: Bluesky ATProto OAuth with session cookies and invite codes
- DB: SQLite via `bun:sqlite` (`data/flint-sync.db`)
- Dev: `cd sync-server && bun run dev`
- Tests: `cd sync-server && bun test`
- See `docs/LEAN-SYNC-SERVER.md` for full architecture details

#### Lean Sync (WebSocket)

Custom one-doc-at-a-time sync replaces automerge-repo's `Repo` on the server (~800MB → ~1MB peak for 2k notes). Speaks the same CBOR wire protocol as automerge-repo — **client requires zero changes**.

- Uses `Automerge.receiveSyncMessage()` / `generateSyncMessage()` low-level API
- Document cache with 30s TTL and WASM memory management (`Automerge.free()`)
- Per-user per-document locks for concurrent access safety
- Sync state memory cache (write-through to SQLite `sync_states` table)
- Real-time fan-out: changes pushed to all other connections for the same user
- Server-initiated sync: after client's initial burst, server pushes docs the client hasn't synced

#### File Sync (REST)

Content-addressed binary file storage for PDFs, EPUBs, images, webpages:

- `PUT /api/files/:fileType/:hash` — Upload with SHA-256 hash verification
- `GET /api/files/:fileType/:hash` — Download with immutable caching
- `GET /api/files/manifest/:vaultId` — List files for a vault
- Conversation JSON storage at `/api/files/conversation/:vaultId/:conversationId`

#### DB Schema (key tables)

- `vault_access` — Maps users to vault document URLs
- `content_doc_access` — Maps users to content document URLs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [disnet/flint-note](https://github.com/disnet/flint-note) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
