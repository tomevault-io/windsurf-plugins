---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## User Preferences

**Important**: The user prefers communication in Chinese (中文). When working on this project:

- Keep conversations in Chinese
- Use Context7 MCP for official documentation lookups when encountering complex/uncertain issues
- Never use `any` type in TypeScript code - maintain strict type safety
- Do not modify files listed in `.gitignore`
- Always run `pnpm lint --fix` after making code changes

## Project Overview

EXIF Gallery Nuxt is a full-stack photo gallery solution deployable on Cloudflare Workers. It features AI-powered image analysis (OpenAI/Gemini), browser-side image compression (JSQuash), complete EXIF metadata management, and edge-native storage using Cloudflare R2 and D1.

**Tech Stack**: Nuxt 4 + Vue 3.5 + NuxtHub + Cloudflare (D1/R2) + UnoCSS + shadcn-vue + Pinia + Drizzle ORM

## Development Commands

```bash
# Development
pnpm dev                    # Start dev server (localhost:3000)
pnpm dev --remote          # Connect to remote Cloudflare resources locally

# Build & Deploy
pnpm build                 # Production build
pnpm preview               # Preview production build
pnpm deploy                # Build and deploy to Cloudflare Workers

# Database
pnpm db:generate           # Generate Drizzle migrations from schema changes

# Code Quality
pnpm lint --fix            # ESLint with @antfu/eslint-config
pnpm typecheck             # Vue + TypeScript type checking

# UI Components
pnpm ui add <component>    # Add shadcn-vue component

# Logs (post-deployment)
pnpm logs                  # Production deployment logs
pnpm logs:preview          # Preview environment logs
```

## Architecture Overview

### Edge-First Data Flow

The application follows a Cloudflare-native architecture where all data lives at the edge:

1. **Upload Flow** (Browser → Server → R2):
   - Browser: Image selected → EXIF extracted via `exifr` → Compressed via JSQuash Web Workers (`app/workers/encode.worker.ts`)
   - Multiple formats generated: original JPEG + optimized WebP + modern AVIF + thumbnail
   - Auto-resize: images with short edge ≥ 2880px are scaled down to 2160px while preserving aspect ratio
   - Server API (`server/api/photos/upload.post.ts`): Receives base64 blobs → Uploads to R2 via NuxtHub → Stores metadata in D1
   - Optional AI analysis: Compressed image sent to OpenAI/Gemini to generate `title`, `caption`, `tags`, `semanticDescription`

2. **Storage Layer**:
   - **D1 (SQLite)**: Photo metadata and relationships (EXIF, tags, associations)
   - **R2 (S3-compatible)**: Binary image blobs served via `/photos/[pathname]` route with aggressive caching (`Cache-Control: public, max-age=31536000, immutable`)
   - Schema: `photos` (main), `tags` (normalized), `photo_tags` (many-to-many junction table)

3. **Query & Display Flow**:
   - API (`server/api/photos/index.get.ts`): Supports pagination (`limit`/`offset`), filtering (by `tag`/`camera`/`lens`/`hidden`), sorting (`takenAt`/`createdAt`)
   - Composable (`app/composables/usePhotos.ts`): `usePhotosInfinite` for infinite scroll, `usePhoto` for single item
   - Pinia store (`app/stores/photos.ts`): Client-side cache with infinite scroll state management

### Database Schema Relationships

The schema uses a modern normalized tag system (migrated from legacy comma-separated `photos.tags` field):

```
photos (1) ←→ (N) photo_tags (N) ←→ (1) tags
```

- `photos.id` (CUID, 8 chars): Primary key for photo records
- `tags.name` (unique): Canonical tag names with `photoCount` denormalized counter
- `photo_tags`: Junction table with cascade delete on both sides
- Indexes: `idx_photos_taken_at`, `idx_photos_hidden`, `idx_tags_photo_count`, `idx_photo_tags_photo_id`, `idx_photo_tags_tag_id`

### AI Integration Architecture

Multi-provider AI configuration managed client-side via localStorage (`app/composables/useAIConfig.ts`):

- Supports OpenAI and Gemini with custom base URL overrides (useful for proxy services)
- Uses `ai-sdk`'s `generateObject` with Zod schema for type-safe structured output
- Image compression before AI analysis: reduces token costs and respects API size limits
- Output: `{ title: string, caption: string, tags: string[], semanticDescription: string }`

### Browser-Side Image Processing

Image compression runs entirely in the browser via Web Workers to avoid server load:

1. **Workers** (`app/workers/`):
   - `decode.worker.ts`: Decodes uploaded images to raw pixel data
   - `encode.worker.ts`: Encodes pixel data to JPEG/WebP/AVIF formats with quality settings

2. **Compression Pipeline** (`app/utils/compress.ts`):
   - Reads file → Extracts EXIF (before compression destroys metadata) → Spawns workers
   - Auto-resize logic: `Math.min(width, height) >= 2880 ? resize to 2160 : keep original`
   - Quality presets: JPEG 0.85, WebP 0.85, AVIF 0.65, Thumbnail 0.7 at 400px

3. **Configuration** (`app/composables/useUploadConfig.ts`):
   - User-controllable: toggle compression, enable/disable specific formats
   - Stored in localStorage, persists across sessions

### Route Patterns & Page Organization

- `/` (index.vue): Home page with featured photos

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wiidede/exif-gallery-nuxt](https://github.com/wiidede/exif-gallery-nuxt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
