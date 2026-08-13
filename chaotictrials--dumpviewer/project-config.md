---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Setup

```bash
npm run install:all          # Install frontend + backend dependencies
```

### Development

```bash
npm run dev                  # Start both frontend (port 5173) and backend (port 3001) with hot reload
npm run dev:frontend         # Frontend only
npm run dev:backend          # Backend only (tsx watch)
```

### Build

```bash
npm run build                # Build frontend (tsc + vite)
npm run build:backend        # Build backend (tsc, copies platform-ids.json to dist/)
```

### Testing

```bash
npm test --prefix backend    # Run backend tests
npm test --prefix frontend   # Run frontend tests
npm run test:all             # Run all tests
```

To run a single test file or suite, use vitest's filter:

```bash
cd backend && npx vitest run src/app.test.ts
cd frontend && npx vitest run src/some.test.ts
```

Frontend vitest runs with `environment: 'node'` (no jsdom), and several modules (e.g. `useHighlighterTheme`) touch `document` at import time — so components cannot be imported in tests. To make component logic testable, extract the pure part into `frontend/src/utils/` and test that (see `utils/configDiff.ts`, extracted from `ConfigViewer`).

### Formatting

```bash
npm run format               # Write (Prettier)
npm run format:check         # Check only
```

## Architecture

This is a React + Express monorepo for viewing [Skyblock Builder](https://github.com/ChaoticTrials/SkyblockBuilder) dump files (`.zip` archives containing a `manifest.json` and game files).

### Key design principle: client-side first

The frontend can parse and display dumps entirely locally — no upload required. The backend is optional and adds: storing dumps by manifest ID, importing from URLs, modpack export, and TTL-based cleanup.

### Backend (`backend/src/`)

- **`index.ts`** — Entry point. Runs `cleanupOldDumps()` on startup and every 24 hours, then starts the HTTP server.
- **`app.ts`** — The entire Express application. Exports `app`, plus utility functions used directly by tests (`isSafeUrl`, `isValidId`, `validateAndExtractManifestId`, `extractManifestInfo`, `parseTtlMs`, `cleanupOldDumps`, `generateDeleteKey`, `resolveDeleteKey`).
- **`platform-api.ts`** — HTTP clients for CurseForge (`curse.moddingx.org`) and Modrinth (`api.modrinth.com`) used for modpack export.
- **`platform-ids.json`** — Static map of mod keys → `{ curseforge, modrinth }` IDs used for modpack generation.

**Storage** is file-based, no database. Each dump is stored as `{DUMPS_DIR}/{uuid}.zip` with a sidecar `{uuid}.meta` (JSON with `expiresAt`/`createdAt` timestamps, `manifestVersion`, the full manifest `versions` object, and optional `hashes`). `GET /api/dumps` backfills `manifestVersion`/`versions` into legacy sidecars on first listing; a persisted `manifestVersion: null` marks an unreadable zip (not retried), while a missing key means "not yet inspected".

**Delete keys** are RSA-2048 tokens: `privateEncrypt(id)` → base64url. The key pair is generated on first startup and persisted in `{DUMPS_DIR}/keys/`. `GET /api/delete/:key` shows an HTML confirmation page (never deletes — safe against link prefetchers); `POST /api/delete/:key` performs the deletion. Both recover the ID by decrypting with the public key — no token storage needed — and share a dedicated rate-limit bucket.

**Auth** is global: a single `AUTH_TOKEN` (env var or `/run/secrets/dumpviewer_token`) gates all write endpoints. The delete-by-key routes are intentionally auth-free. Brute-force protection: failed auth attempts are delayed by `AUTH_FAIL_DELAY_MS` (default 500 ms) and counted by a dedicated `authFailureLimiter` (only 401 responses count, via `requestWasSuccessful`; `AUTH_FAIL_LIMIT` failures per 15 min → IP lockout). Every endpoint sits behind a rate-limit bucket except `/health`; `GENERAL_RATE_LIMIT` and `MODPACK_RATE_LIMIT` tune the read and modpack buckets.

**SSRF protection** for URL imports is two-layered: `isSafeUrl()` string checks, plus `safeLookup()` — a validating DNS lookup wired into an undici `Agent` so every connection (including redirect hops) rejects hostnames resolving to private/loopback/link-local addresses (DNS rebinding). Import fetches therefore go through `undici`'s `fetch`, not the global one; `app.test.ts` mocks `undici.fetch` by delegating to `globalThis.fetch` so `vi.stubGlobal('fetch', ...)` still intercepts imports.

**Zip handling**: always check `entry.header.size` (declared uncompressed size) against a limit before calling `getData()` on untrusted zips — decompression-bomb protection (`MAX_MANIFEST_ENTRY_BYTES`, `MAX_OVERRIDE_ENTRY_BYTES`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChaoticTrials/DumpViewer](https://github.com/ChaoticTrials/DumpViewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
