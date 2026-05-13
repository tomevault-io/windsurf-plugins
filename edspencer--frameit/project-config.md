---
trigger: always_on
description: FrameIt is a lightweight, open-source image generator for creating beautiful title images—thumbnails, OG images, and title cards—across multiple platforms. Built with React, TypeScript, and Tailwind CSS.
---

# FrameIt Development Guide

FrameIt is a lightweight, open-source image generator for creating beautiful title images—thumbnails, OG images, and title cards—across multiple platforms. Built with React, TypeScript, and Tailwind CSS.

## Quick Start

```bash
# Install dependencies
pnpm install

# Start development server (runs on http://localhost:4321)
pnpm dev

# Build for production (Astro SSG output to dist/)
pnpm build

# Preview production build
pnpm preview

# Typecheck Astro and TypeScript files
pnpm typecheck
```

**Note:** The development server runs on port 4321 (Astro's default), not port 5173 (Vite's default).

## Technology Stack

- **Framework**: Astro 5 with React islands (React 19 RC)
- **Language**: TypeScript 5 (strict mode)
- **Styling**: Tailwind CSS 3 with @tailwindcss/typography for prose
- **Content**: Astro Content Collections with Zod schema
- **Deployment**: Vercel (with @astrojs/vercel adapter)
- **Canvas**: HTML5 Canvas API (UI) + @napi-rs/canvas (API)
- **API**: Vercel Serverless Functions for programmatic generation
- **Analytics**: PostHog (privacy-compliant) + Vercel Analytics

GitHub repo: https://github.com/edspencer/frameit

## Project Structure

```
src/
├── pages/                        # Astro pages (file-based routing)
│   ├── index.astro               # Homepage with ThumbnailGenerator island
│   └── guides/
│       ├── index.astro           # Guide listing page
│       └── [...slug].astro       # Dynamic guide pages (content collection)
├── layouts/
│   └── BaseLayout.astro          # Shared layout with meta tags, fonts, analytics
├── content/
│   ├── config.ts                 # Content collection schema (Zod validation)
│   └── guides/                   # Guide markdown files with frontmatter
│       ├── why-og-images-matter.md
│       ├── og-image-technical-specs.md
│       ├── og-image-design-principles.md
│       ├── og-image-design-patterns.md
│       ├── og-image-automation.md
│       ├── og-image-testing-validation.md
│       ├── og-image-best-practices-checklist.md
│       └── og-image-common-mistakes.md
├── components/
│   ├── ThumbnailGenerator.tsx    # Main component (React island with client:only)
│   ├── CanvasPreview.tsx         # Canvas rendering wrapper
│   ├── ControlPanel.tsx          # Control UI container
│   ├── Navigation.astro          # Site navigation (Astro component)
│   ├── Footer.astro              # Site footer (Astro component)
│   ├── GuideNavigation.astro     # Prev/next guide navigation
│   ├── TableOfContents.astro     # Auto-generated TOC from headings
│   └── ... (other React components)
├── lib/
│   ├── constants.ts              # Platform presets and backgrounds
│   ├── constants/layouts.ts      # Layout definitions
│   ├── types.ts                  # TypeScript interfaces
│   ├── canvas-utils.ts           # Canvas drawing utilities
│   ├── layout-renderer.ts        # Layout rendering engine
│   ├── posthog.ts                # Analytics tracking functions
│   └── ui-state.ts               # State management utilities
├── hooks/
│   └── useExampleFromUrl.ts      # URL query parameter handling
└── index.css                     # Global styles (Tailwind imports)

api/
└── generate.ts                   # Serverless API for image generation

public/
├── frameit-logo.png              # Site logo
├── frameit-icon.png              # Favicon
├── open-graph.png                # Default OG image
└── robots.txt                    # Search engine directives

astro.config.ts                   # Astro configuration with integrations
```

## Key Features

- **Multiple Platform Presets**: YouTube, YouTube Shorts, Twitter/X, TikTok, Square
- **Real-time Preview**: Live canvas updates as you customize
- **Independent Color Controls**: Separate colors for heading and subheading
- **Background Options**: Multiple gradient backgrounds
- **Logo Opacity**: Adjustable BragDoc logo opacity
- **Download & Copy**: Export as PNG or copy to clipboard
- **Persistent State**: Settings saved to localStorage

## State Management

`ThumbnailGenerator.tsx` manages all application state:
- `selectedPreset`: Current platform dimensions
- `title` / `subtitle`: Text content
- `titleColor` / `subtitleColor`: Independent text colors
- `selectedBackground`: Background image URL
- `logoOpacity`: Logo opacity (0-1)

All state is persisted to localStorage via `saveConfigToStorage()` and restored on page load via `loadConfigFromStorage()`.

## Canvas Rendering

FrameIt uses a layout-based rendering system via `LayoutRenderer` class ([src/lib/layout-renderer.ts](src/lib/layout-renderer.ts)):
- **Layout System**: JSON-defined layouts with text, image, and overlay elements
- **Positioning**: Supports percentage-based, pixel-based, and auto positioning
- **Anchor Points**: 9-point anchor system (top-left, center, bottom-right, etc.)
- **Text Wrapping**: Automatic word wrapping via `wrapText()` utility
- **Gradient Backgrounds**: Linear gradients via `drawGradientBackground()`

The same `LayoutRenderer` is used for both UI preview and API generation, ensuring 1:1 parity.

## API Endpoint


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [edspencer/frameit](https://github.com/edspencer/frameit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
