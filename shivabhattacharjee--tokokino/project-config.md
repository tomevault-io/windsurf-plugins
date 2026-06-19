---
trigger: always_on
description: **Tokokino** is a browser-based screenshot beautifier. Users drop in a screenshot, style it with backgrounds, shadows, device frames, text layers, and annotations, then export as PNG/JPEG/WebP or share a public link. The app is fully client-side for editing; the server handles auth, share/draft uploads to Cloudflare R2, and metadata/view tracking in Cloudflare D1 via OpenNext Cloudflare.
---

# CLAUDE.md — Tokokino 

## Project overview

**Tokokino** is a browser-based screenshot beautifier. Users drop in a screenshot, style it with backgrounds, shadows, device frames, text layers, and annotations, then export as PNG/JPEG/WebP or share a public link. The app is fully client-side for editing; the server handles auth, share/draft uploads to Cloudflare R2, and metadata/view tracking in Cloudflare D1 via OpenNext Cloudflare.

**Stack:**
- Next.js 15.5.18 (App Router) + React 19.2, Turbopack in dev
- OpenNext Cloudflare (`@opennextjs/cloudflare` 1.19) + Wrangler 4 for Cloudflare Workers deployment
- Zustand 5 for all editor state, with undo/redo and IndexedDB persistence
- Tailwind CSS v4 + shadcn components + Radix UI primitives
- better-auth (email + Google OAuth), Cloudflare D1 via Drizzle, Cloudflare R2 via AWS S3 SDK
- `html-to-image` for canvas capture, `motion` for animation, `@dnd-kit` for drag-and-drop
- Zod v4 (`zod/v4`) for input validation
- TypeScript strict mode

---

## Dev commands

```bash
pnpm dev          # starts Next.js with Turbopack
pnpm build        # OpenNext Cloudflare production build
pnpm build:next   # raw next build used by OpenNext
pnpm preview      # OpenNext Cloudflare build + local preview
pnpm deploy       # OpenNext Cloudflare build + deploy
pnpm typecheck    # tsc --noEmit (run before committing)
pnpm lint:fix     # ESLint auto-fix
pnpm format       # Prettier on all .ts/.tsx
```

Asset build scripts (run once after adding overlays/backgrounds):
```bash
pnpm build:thumbs                 # overlay thumbnails
pnpm build:backgrounds            # background thumbnails
```

---

## Directory structure

```
/app                      Next.js App Router pages, API routes, and metadata routes
  /app/page.tsx           Main editor page (EditorLayout)
  /app/share/page.tsx     User's share history
  /app/layout.tsx         App shell (providers)
  /api/share/route.ts     POST: create share link
  /api/auth/[...all]      better-auth handler
  /api/export/image       CORS proxy for external images
  /api/unsplash/*         Unsplash search + download
  /login/                 Auth pages
  /share/[id]/            Public share view
  /terms/                 Terms of service
  sitemap.ts               Generated sitemap.xml
  robots.ts                Generated robots.txt
  /llms.txt/route.ts       AI crawler summary endpoint

/components/editor/       All editor UI (see Editor Components below)
/components/ui/           shadcn component library
/lib/editor/              Core editor logic
  store.tsx               Zustand store — all state & actions
  state-types.ts          All TypeScript types
  export.ts               Image capture & export
  css-utils.ts            CSS generation (shadows, filters, backgrounds)
  color-utils.ts          Color sampling & gradient generation
  fonts.ts                Google Fonts catalogue (100+ fonts)
  presets.ts              Gradient/solid/overlay presets
  present-presets.ts      Multi-screenshot layout presets + single tilt presets
  screenshot-layout.ts    Row layout algorithm for multi-screenshot
  value-schemas.ts        Zod schemas for all numeric inputs
  types.ts                Misc types
/lib/
  auth.ts                 better-auth server instance
  auth-client.ts          Client-side auth hooks
  env.ts                  Environment variable validation
  d1.ts                   Cloudflare D1 + Drizzle entrypoint via OpenNext context
  share.ts                Share URL helpers, UUID validation
  share-db.ts             D1 share CRUD + view tracking
  draft-db.ts             D1 draft metadata CRUD
  preset-db.ts            D1 custom preset CRUD
  share-storage.ts        R2 share image upload/download
  draft-storage.ts        R2 draft state + thumbnail storage
  r2-client.ts            R2 S3-compatible client
  browser-frame.ts        Browser frame constants (Safari/Chrome/Arc)
/hooks/
  use-floating-toolbar-rect.ts  Toolbar positioning hook
```

---

## Editor state — Zustand store (`lib/editor/store.tsx`)

The entire editor state lives in one Zustand store with temporal middleware for undo/redo.

### Top-level state shape

```ts
{
  // Editor UI
  activeTool: EditorTool            // "pointer"|"crop"|"text"|"arrow"|"position"|"layers"|"enhance"
  aspect: AspectState               // { id, w, h } — canvas aspect ratio
  canvasZoom: number                // editor viewport zoom (not the screenshot scale)
  annotation: Annotation            // current annotation tool state

  // Preview / bulk edit
  isPreviewMode: boolean
  isPreviewAutoScroll: boolean
  previewAutoScrollDelay: number
  previewAnimation: "slide"|"fade"|"zoom"|"flip"
  bulkEditMode: boolean
  bulkCanvasDragging: boolean
  bulkViewportZoom: number

  // Layout preset tracking
  presetTab: "single"|"multi"
  activeLayoutPresetId: string | null   // multi-screenshot preset
  activeSinglePresetId: string | null   // single-screenshot tilt preset

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ShivaBhattacharjee/Tokokino](https://github.com/ShivaBhattacharjee/Tokokino) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
