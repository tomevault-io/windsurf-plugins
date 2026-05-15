---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a TypeScript monorepo (NestJS backend + React/Mantine frontend) managed with Turbo. Always verify TypeScript compilation (`tsc --noEmit`) after making changes to DTOs, API decorators, or shared types.

Litara is a self-hosted ebook library manager. It is a monorepo managed with **Turborepo** and **npm workspaces**, containing two apps and one package:

- `apps/api` — NestJS backend (REST API + library scanner)
- `apps/web` — React + Vite frontend
- `apps/mobile` — React Native / Expo mobile app. Audiobook playback uses `react-native-track-player` (singleton service architecture). Playback service, queue builder, and progress saver live under `apps/mobile/src/services/playback/`. The entry point is `apps/mobile/index.ts` which registers the RNTP playback service before loading the expo-router entry.
- `packages/mobi-parser` (`@litara/mobi-parser`) — Wraps `@lingo-reader/mobi-parser` to extract metadata from `.mobi`, `.azw`, and `.azw3` files. Exposes a single `extractMobiMetadata(filePath)` function.
- `packages/book-types` (`@litara/book-types`) — **Single source of truth for all enrichable book metadata fields.** Exports `BookMetadataFields` (canonical interface) and `MetadataResult` (extends it with `categories`). **All metadata field additions/renames must start here** and propagate to `MetadataResultDto` and `UpdatePendingBookDto` (both `implements BookMetadataFields`).

## Commands

### Monorepo (run from root)

```bash
npm run dev       # Start all apps in watch mode (via Turborepo)
npm run build     # Build all apps
npm run lint      # Lint all apps
npm run format    # Prettier format all TS/TSX/MD files
```

### API (`apps/api`)

```bash
npm run dev           # Watch mode (nest start --watch)
npm run build         # nest build → dist/
npm run start:prod    # node dist/main
npm run test          # Jest unit tests
npm run test:e2e      # Jest e2e tests (uses test/jest-e2e.json)
npm run test:watch    # Jest in watch mode
npm run lint          # ESLint with auto-fix
```

Run a single test file:

```bash
cd apps/api && npx jest src/path/to/file.spec.ts
```

### Web (`apps/web`)

```bash
npm run dev       # Vite dev server
npm run build     # tsc -b && vite build
npm run preview   # Preview production build
npm run lint      # ESLint
```

### Database

```bash
# Start PostgreSQL (required before running the API)
docker-compose up -d

# Create a new migration after editing prisma/schema.prisma
cd apps/api && npx prisma migrate dev --name <migration-name>

# Apply migrations (the API does this automatically on startup via execSync)
cd apps/api && npx prisma migrate deploy

# Open Prisma Studio
cd apps/api && npx prisma studio
```

## Architecture

### API (`apps/api`)

NestJS app with URI versioning (`/api/v1/...`). Swagger UI is served at `/docs`.

**Modules:**

- `DatabaseModule` — Wraps PrismaClient using the `@prisma/adapter-pg` driver (Prisma 7 driver adapters). Runs `prisma migrate deploy` automatically on startup.
- `AuthModule` — JWT + Passport local strategy. JWT tokens expire in 60 minutes. Secret read from `JWT_SECRET` env var.
- `UsersModule` — User management with bcrypt password hashing.
- `LibraryModule` — Contains `LibraryScannerService`, which on startup does a full scan of watched folders using `fast-glob`, then switches to continuous file watching with `chokidar`. Supported formats: `epub`, `mobi`, `azw`, `azw3`, `cbz`, `pdf`, `fb2`, `cbr`, `cb7`. EPUB metadata is extracted via `epub2`, `.mobi`, `.azw`, `.azw3` are parsed via `mobi-parser` (internal package). CBZ parsed by `cbz-parser` (internal package). Other formats fall back to filename-based metadata.

**Prisma schema** is in `apps/api/prisma/schema.prisma`. Key models: `User`, `Library`, `Book`, `Author`, `BookAuthor`, `BookFile`, `WatchedFolder`, `ReadingProgress`, `Annotation`, `Shelf`, `SmartShelfRule`, `Task`, `Log`, `ServerSettings`.

**Library scanning flow:**

1. On init, ensures a "Default Library" and a `WatchedFolder` exist (path from `EBOOK_LIBRARY_PATH` env var, or `ebook-library/` sibling directory).
2. Full scan with `fast-glob` to import existing files.
3. Chokidar watcher for incremental adds/removes.
4. Duplicate detection via SHA-256 file hash (`fileHash` on `BookFile`).

### Web (`apps/web`)

React 19 + Vite SPA using **Mantine** for UI components and **React Router v7** for routing.

- Auth is JWT-based; token stored in `localStorage`.
- All API calls go through a shared `axios` instance (`src/utils/api.ts`) with base URL `/api/v1` (relative; overridable via `VITE_API_URL` env var). The interceptor auto-attaches the Bearer token and redirects to `/login` on 401.
- Routes: `/login` (public), `/` (protected, Dashboard).

### Environment Variables

| Variable               | App | Description                                                                                                              |
| ---------------------- | --- | ------------------------------------------------------------------------------------------------------------------------ |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [litara-app/litara](https://github.com/litara-app/litara) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
