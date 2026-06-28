---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

Immich Folio is a self-hosted photography portfolio that acts as a secure reverse proxy between visitors and a private Immich instance. It serves albums via an image proxy so the Immich server and API key are never exposed to the public internet. Asset UUIDs are AES-256 encrypted into opaque URL tokens.

## Commands

```bash
npm run dev          # Start dev server at http://localhost:3000
npm run build        # Production build (must pass before any PR)
npm run lint         # ESLint
npm run format       # Prettier (write)
npm run format:check # Prettier (check only)
npx tsc --noEmit     # TypeScript type-check (must be 0 errors before any PR)

# Unit tests (Vitest, lib/__tests__/ only)
npm run test         # Watch mode
npm run test:unit    # Single run
npm run test:coverage

# E2E tests (Playwright, requires dev server running or starts it automatically)
npm run test:e2e
```

**Formatting caution:** Running `npm run format` at the repo root applies changes to the entire codebase and pollutes diffs. Prefer formatting only modified files.

## Architecture

### Configuration loading (`lib/config/`)

Configuration is loaded once at startup (cached in a module-level singleton, re-read in dev mode per request).

- `lib/env.ts` — parses and validates all environment variables into a typed `Env` object. All env access in the codebase must go through this module.
- `lib/config/schema.ts` — TypeScript types for `AppConfig`, `GalleryYaml`, `SettingsYaml`, `SubpageConfig`, `GridConfig`, `ThemeConfig`, etc., plus the `slugify()` utility.
- `lib/config/index.ts` — `getConfig()`: reads `content/gallery.yaml` and `content/settings.yaml`, merges them into a single `AppConfig`. Also exports `buildSubpageGrid()` and re-exports from schema/theme.
- `lib/config/theme.ts` — six built-in theme presets (`studio`, `minimal`, `editorial`, `classic`, `noir`, `monograph`) and `resolveTheme()` which merges partial overrides over a preset.

`getConfig()` returns a `needsSetup: true` dummy config (instead of throwing) when `gallery.yaml` or credentials are missing — this lets the app render a `SetupScreen` instead of crashing.

### Content files (`content/`)

- `content/gallery.yaml` — gallery structure: hero asset IDs, standalone albums, subpages (with optional sections, passwords, per-subpage grid overrides). Use `gallery.yaml.example` as reference.
- `content/settings.yaml` — site-wide settings: title, subtitle, theme, grid defaults, footer, legal, map, transitions, SEO. Use `settings.yaml.example` as reference.
- `content/about.md` — Markdown with frontmatter for the about page (portrait, name, location, gear).

### Immich client (`lib/immich.ts`)

Singleton `ImmichClient` class exported as `immich`. All Immich API calls are server-side only. Key design points:

- **Album allowlist** — `getAlbums()` fetches `?shared=true` albums but only returns IDs listed in `config.albums`. Requests for unlisted albums are silently rejected.
- **Request coalescing** — pending promises are stored in `Map<id, Promise>` fields to deduplicate concurrent requests for the same album/asset (important for `Promise.all` calls in grids).
- **In-memory LRU cache** (`lib/cache.ts`) — 200-entry LRU, configurable TTL via `CACHE_TTL`. Cache keys: `albums-list`, `album-<id>`, `asset-<id>`.
- **Image streaming** — `streamAsset()` proxies binary responses; never loads the full image into memory.

### Asset token security (`lib/tokens.ts`)

`encodeAssetId(uuid)` / `decodeAssetId(token)` — AES-256-GCM with a deterministic IV derived from the asset ID (same UUID → same token, enabling browser caching). Token format: `v2:<base64url(iv+authTag+ciphertext)>`. The encryption key is derived from `AUTH_SECRET` via SHA-256. Legacy CBC tokens are still decoded for backward compatibility.

### API routes (`app/api/`)

| Route | Purpose |
|---|---|
| `GET /api/image/[id]` | Image proxy — decodes token, rate-limits, streams from Immich |
| `GET /api/exif/[id]` | EXIF data for lightbox panel |
| `POST /api/auth` | Password submission → sets `HttpOnly` cookie |
| `GET /api/og` | Dynamic OG image generation (rate-limited) |
| `GET /api/map` | Aggregated GPS coordinates for map view |
| `GET /api/health` | Health check |
| `GET/POST/DELETE /api/admin/auth` | Admin login, session check, logout |
| `GET/PUT /api/admin/gallery` | Read/write `gallery.yaml` |
| `GET/PUT /api/admin/settings` | Read/write `settings.yaml` |
| `GET /api/admin/albums` | Browse all shared Immich albums (admin-only) |
| `POST /api/admin/reload` | Invalidate config + Immich cache |

The image proxy maps requested pixel widths (`?w=`) to Immich size tiers: `≤250px→thumbnail`, `≤1440px→preview`, `>1440px→original`.

### Routing (`app/[...path]/page.tsx`)

Single catch-all route handles three cases:
1. `/[subpage-slug]/[album-slug]` — renders album detail with back-link to subpage
2. `/[subpage-slug]` — if subpage has >1 album, renders `SubpageGridView`; if exactly 1 album, renders `AlbumDetailView` directly
3. `/[album-slug]` — standalone album detail

All pages are `dynamic = 'force-dynamic'` (no SSG; requires live Immich).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ralksta/immich-folio](https://github.com/ralksta/immich-folio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-28 -->
