---
trigger: always_on
description: ChronoFrame is a high-performance photo management and gallery application built with Nuxt 4, featuring WebGL-accelerated image viewing, comprehensive EXIF processing, and multi-storage backend support.
---

# ChronoFrame AI Coding Guidelines

ChronoFrame is a high-performance photo management and gallery application built with Nuxt 4, featuring WebGL-accelerated image viewing, comprehensive EXIF processing, and multi-storage backend support.

## Architecture Overview

### Core Components

- **Frontend**: Nuxt 4 app with Vue 3, TypeScript, and TailwindCSS
- **Backend**: Nitro server with SQLite (Drizzle ORM) and multi-provider storage
- **WebGL Package**: Custom `@chronoframe/webgl-image` for hardware-accelerated viewing
- **Processing Pipeline**: Async photo processing with EXIF, thumbnails, and geolocation

### Key Directories

- `app/`: Nuxt 4 application (components, pages, composables)
- `server/`: API routes and backend services
- `packages/webgl-image/`: Standalone WebGL image viewer package
- `shared/`: Type definitions shared between client/server

## Development Workflow

### Essential Commands

```bash
# Development with dependency building
pnpm dev:deps

# Build WebGL package only
pnpm build:deps

# Database operations
pnpm db:generate  # Generate migrations
pnpm db:migrate   # Apply migrations

# Production build
pnpm build
```

### Monorepo Structure

This is a pnpm workspace with the WebGL package as a local dependency. Always use `pnpm dev:deps` for development to ensure the WebGL package builds alongside the main app.

## Storage Architecture

### Multi-Provider System

Storage providers are abstracted through `server/services/storage/interfaces.ts`:

- **S3**: AWS S3-compatible storage (primary)
- **HubR2**: Cloudflare R2 via NuxtHub
- **GitHub**: Git-based storage (experimental)

### Key Pattern

All storage operations go through `useStorageProvider(event)` in API routes. The provider is configured via `NUXT_STORAGE_PROVIDER` environment variable.

## Photo Processing Pipeline

### Async Processing Pattern

Photos are processed via `execPhotoPipelineAsync()` in `server/services/photo/pipeline-async.ts`:

1. **Preprocessing**: HEIC conversion to JPEG, buffer management
2. **Metadata**: Sharp processing for dimensions/format
3. **Thumbnails**: WebP generation with ThumbHash
4. **EXIF**: Comprehensive metadata extraction via exiftool-vendored
5. **Geolocation**: Reverse geocoding for GPS coordinates
6. **LivePhoto**: Video companion file detection and processing

### Critical Implementation Details

- Use `setImmediate()` for non-blocking async operations
- Always handle HEIC → JPEG conversion for Apple photos
- EXIF processing requires temporary file writes (see `server/services/image/exif.ts`)
- Thumbnails are stored as separate WebP files with hash compression

## Component Patterns

### Vue Components

- All components are auto imported via Nuxt (eg. `/app/components/photo/PhotoItem.client.vue` can be used directly as `<PhotoPhotoItem />`)

### Vue Composables

- `usePhotos()`: Central photo data management with injection/provide pattern
- `usePhotoFilters()`: Client-side filtering with reactive state
- `useLivePhotoProcessor()`: WebGL-based MOV to MP4 conversion
- `useWebGLWorkState()`: Performance monitoring for WebGL operations

### Masonry Grid System

The `app/components/masonry/Root.vue` implements a CSS-based masonry layout:

- Auto-responsive column counts based on viewport
- Intersection Observer for performance and date range tracking
- Background LivePhoto processing for visible items only

### WebGL Integration

Register WebGL components via plugin (`app/plugins/chrono-webgl-image.ts`). The package provides hardware-accelerated zooming and panning for large images.

## Database Operations

### Core Database Pattern

**ALWAYS use `useDB()` from `server/utils/db.ts` for all database operations:**

```typescript
import { useDB, tables, eq } from '~~/server/utils/db'

// Get all photos
const photos = await useDB().select().from(tables.photos)

// Get specific photo
const photo = await useDB()
  .select()
  .from(tables.photos)
  .where(eq(tables.photos.id, photoId))
  .get()

// Insert new photo
await useDB().insert(tables.photos).values(photoData)

// Update photo
await useDB()
  .update(tables.photos)
  .set({ title: 'New Title' })
  .where(eq(tables.photos.id, photoId))
```

### Database Configuration

- Uses **better-sqlite3** with Drizzle ORM
- Database file: `data/app.sqlite3`
- Schema exports types: `User`, `Photo` from `useDB`
- Import patterns: `eq`, `and`, `or`, `sql` from the same file

### Photo Model Schema

Key fields in `server/database/schema.ts`:

- `storageKey`: Original file path in storage
- `originalUrl`: Public URL (may point to JPEG version for HEIC)
- `thumbnailUrl`/`thumbnailHash`: WebP thumbnail with ThumbHash
- `exif`: Full EXIF JSON (typed as `NeededExif`)
- `latitude`/`longitude`/`city`/`country`: Extracted geolocation
- `isLivePhoto`: Boolean with optional video companion file

### Migration Pattern

Use Drizzle migrations for schema changes. Initial user setup is handled in migration files.

## API Conventions

### Authentication

All photo management APIs require `await requireUserSession(event)` using nuxt-auth-utils with GitHub OAuth.

### Upload Flow

1. POST `/api/photos` → Get presigned URL

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HoshinoSuzumi/chronoframe](https://github.com/HoshinoSuzumi/chronoframe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
