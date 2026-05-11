---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

WALL·E Gallery is a high-performance wallpaper gallery app that displays images from **any public GitHub repository**. It supports multiple wallpaper sources (engines) and implements aggressive performance optimizations to handle heavy images efficiently.

**Tech Stack:** React 19 + TypeScript + Vite + Tailwind CSS v3

## Development Commands

```bash
# Start development server (localhost:5173)
npm run dev

# Type check and build for production
npm run build

# Preview production build
npm run preview

# Lint code
npm run lint
```

## Docker Deployment

```bash
# Build and run with Docker Compose
docker-compose up -d --build

# View logs
docker logs wall-e-gallery

# Stop
docker-compose down
```

Uses Caddy as web server (configured in `Caddyfile`). Exposes port 80.

## Architecture

### Engine System (Multi-Source Support)

The app supports multiple wallpaper sources called "engines". Each engine represents a different GitHub repository:

**Engine Storage:**
- **Default engines**: Stored in `src/data/default-engines.json` (3 built-in repositories)
- **Custom engines**: User-added, stored in localStorage under `'wallpaper-engines'`
- **Active engine**: Tracked in localStorage under engine metadata

**Engine Properties:**
```typescript
{
  id: string;                    // Unique identifier
  name: string;                  // Display name (e.g., "<owner>/<repoName>")
  repoOwner: string;             // GitHub username
  repoName: string;              // Repository name
  branch: string;                // Branch to fetch from (usually "main")
  treeSha: string;               // Git commit SHA (40-char hex)
  excludedFolders: string[];     // Folders to ignore (e.g., [".github", "docs"])
  imageExtensions: string[];     // Supported formats (e.g., [".png", ".jpg"])
  isDefault: boolean;            // True for built-in engines
  avatarUrl?: string;            // GitHub user avatar
  createdAt?: number;            // Timestamp for custom engines
}
```

**Adding New Default Engines:**
Update `src/data/default-engines.json` with the new engine configuration. Fetch tree SHA using:
```bash
curl -s https://api.github.com/repos/{owner}/{repo}/branches/{branch} | grep '"sha"' | head -1
```

### Data Flow & Caching Strategy

The app fetches repository trees once per engine and caches them for the session:

1. **GitHub API** (`src/lib/github-api.ts:fetchRepoTree()`) - Fetches recursive tree structure
2. **Per-engine cache** (`engineCache` Map) - Each engine's data cached separately
3. **Request deduplication** (`inflightRequests` Map) - Prevents concurrent duplicate API calls
4. **Client-side operations** - All filtering, searching, sorting happen on cached data

**Cache Management:**
- Cache cleared when switching engines
- Each engine can maintain its own cache simultaneously
- No automatic refresh (requires page reload or engine re-selection)

### Performance Optimization System

The app uses a **three-tier image loading strategy** to handle heavy wallpapers:

#### 1. Image Proxy Service (wsrv.nl)
- **Location:** `src/lib/github-api.ts:getThumbnailUrl()`
- Generates WebP thumbnails on-the-fly (from ~10MB originals to ~50KB)
- All gallery images use thumbnails only
- **Three sizes available:**
  - Small: 300px width, 75% quality
  - Medium: 400px width, 80% quality (default)
  - Large: 600px width, 85% quality
- User can toggle between sizes (persisted to localStorage as `'thumbnailSize'`)

#### 2. Progressive Loading Components
- **ProgressiveImage** (`src/components/ProgressiveImage.tsx`) - Shows skeleton → thumbnail with blur → sharp
- **VirtualMasonryGallery** (`src/components/VirtualMasonryGallery.tsx`) - Gallery uses thumbnails only, column count adjusts based on thumbnail size
- **OptimizedImageModal** (`src/components/OptimizedImageModal.tsx`) - Loads full resolution only when modal opens (lazy-loaded)

#### 3. Code Splitting & Lazy Loading
Heavy components are lazy-loaded with React.lazy() and Suspense:
- **OptimizedImageModal** - Loaded when user clicks an image
- **EnginesModal** - Loaded when user opens Settings

Skeleton fallbacks during lazy load:
- **GallerySkeleton** (`src/components/GallerySkeleton.tsx`) - Grid placeholder during initial load
- **ModalSkeleton** (`src/components/ModalSkeleton.tsx`) - Modal loading fallback
- **ImageCardSkeleton** (`src/components/ImageCardSkeleton.tsx`) - Single image placeholder

Vite manual chunks configured in `vite.config.ts` for `react-vendor` and `icons`.

#### 4. Rendering Strategy
- **Infinite scroll** (`src/hooks/useInfiniteScroll.ts`) - Loads 20 images per batch
- **Intersection Observer** - Images load 200px before entering viewport
- **Flexbox masonry** - Manual round-robin distribution prevents scrambling on new loads

**Result:** 99.5% bandwidth reduction, 90% memory reduction (see PERFORMANCE.md for details)

### Component Architecture

#### Core Layout
- **App.tsx** - Main orchestrator, manages state and data flow
- **Sidebar.tsx** - Category navigation + theme toggle
- **VirtualMasonryGallery.tsx** - Main gallery view (thumbnails only)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amitray007/wall-e](https://github.com/amitray007/wall-e) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
