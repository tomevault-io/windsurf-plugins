---
trigger: always_on
description: pnpm --filter web dev
---

# AGENTS

## Commands

### Development Commands

```bash
# Start development server (runs both web and SSR)
pnpm dev

# Start only web development server
pnpm --filter web dev

# Start only SSR development server
pnpm --filter @afilmory/ssr dev

# Build production version
pnpm build

# Build manifest from storage (generates photo metadata)
pnpm run build:manifest

# Force rebuild all photos and metadata
pnpm run build:manifest -- --force

# Force regenerate thumbnails only
pnpm run build:manifest -- --force-thumbnails

# Force regenerate manifest only
pnpm run build:manifest -- --force-manifest
```

### Database Commands (SSR app)

```bash
# Generate database migrations
pnpm --filter @afilmory/ssr db:generate

# Run database migrations
pnpm --filter @afilmory/ssr db:migrate
```

### Code Quality Commands

```bash
# Lint and fix code
pnpm lint

# Format code
pnpm format

# Type check (web app)
pnpm --filter web type-check
```

## Architecture

The project employs a sophisticated, modular architecture that separates concerns across different applications and packages, enabling independent development, deployment, and scaling.

### Core Components

The project is divided into four main applications:

1.  **`apps/web` - Standalone Frontend SPA**
    *   **Description**: A pure client-side application built with React, Vite, and TypeScript. It can be deployed independently as a static website and is fully functional on its own.
    *   **UI/Design**: Features a modern, interactive, and user-centric UI. It utilizes a "Glassmorphic Depth Design System" for components like modals, toasts, and floating panels, creating a sense of visual hierarchy through layered transparency and subtle color accents. The design is geared towards a rich end-user experience for photo browsing and visualization.
    *   **Server Integration**: It can operate in two modes:
        *   **Standalone**: Functions without a server, using a pre-built `photos-manifest.json` file.
        *   **Server-Connected**: When a global variable like `window.__MANIFEST__` is detected, it unlocks enhanced features. This injection is handled by either `apps/ssr` (from a static file) or `be/apps/core` (from the database).

2.  **`apps/ssr` - Next.js Wrapper for SEO & Prerendering**
    *   **Description**: A Next.js application that acts as a transparent proxy for the `apps/web` SPA. Its primary role is to enhance the frontend with server-side capabilities for performance and discoverability, rather than serving as a full-fledged backend. It injects the manifest from a static JSON file.
    *   **Key Features**:
        *   **OG (Open Graph) Rendering**: Dynamically generates social media preview cards for shared links.
        *   **SEO Metadata Injection**: Injects dynamic SEO tags into the HTML for better search engine visibility.
        *   **SSR for Shared Pages**: Server-renders specific pages to provide fast initial load times.

- **`be/apps/core`**: The complete backend server (Hono) for real-time data. For a detailed breakdown of its architecture, see `be/apps/core/AGENTS.md`.
- **`be/apps/dashboard`**: The administration panel for the backend, using a linear, data-first admin aesthetic (crisp frames, subtle gradients). See `be/apps/dashboard/AGENTS.md` for full UI guidelines.

### Monorepo Structure

This is a pnpm workspace with multiple applications and packages:

- `apps/web/` - Main frontend React application (Vite + React 19 SPA).
- `apps/ssr/` - Next.js 15 application serving as an SPA host and dynamic SEO/OG generator.
- `be/apps/core/` - The complete backend server (Hono) for real-time data.
- `be/apps/dashboard/` - The administration panel for the backend (linear, data-first admin look).
- `packages/builder/` - Photo processing and manifest generation tool.
- `packages/webgl-viewer/` - High-performance WebGL-based photo viewer component.
- `packages/data/` - Shared data access layer and PhotoLoader singleton.
- `packages/components/` - Reusable UI components shared across apps.
- `packages/ui/` - Core UI elements and design system components.
- `packages/hooks/` - Shared React hooks.
- `packages/utils/` - Utility functions.

### Next.js as SPA Host & SEO Provider

**Dual Server Architecture (for `apps/ssr`)**:

- **Development Mode**: `apps/ssr/src/app/[...all]/route.ts` catches all SPA routes and serves `index.html` with injected manifest data from the static JSON file.
- **Production Mode**: Next.js serves pre-built Vite SPA assets while providing dynamic OG image generation.

**Dynamic SEO Implementation**:

- `apps/ssr/src/index.html.ts` - Pre-compiled HTML template with manifest data injected as `window.__MANIFEST__`.
- Dynamic OG images generated per photo via Next.js API routes (`/og/[photoId]/route.ts`).
- HTML meta tags dynamically replaced for social media sharing.

### Configuration Architecture

**Two-Layer Configuration System**:

1. **Builder Config** (`builder.config.ts`) - **Infrastructure/Processing Layer**:

   - Controls photo processing, storage connections, and build performance.
   - Handles remote git repository sync for manifest/thumbnails.
   - Configures multi-process/cluster processing for large photo sets.

2. **Site Config** (`site.config.ts` + `config.json`) - **Presentation/Content Layer**:
   ```typescript
   {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Afilmory/afilmory](https://github.com/Afilmory/afilmory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
