---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a personal portfolio website built with Next.js 16.0.10 and React 19.2.3. The site is designed for serverless deployment (Vercel) and includes Progressive Web App (PWA) capabilities with offline support via @ducanh2912/next-pwa.

## Development Commands

### Setup
```bash
npm install
```

### Environment Variables
Create a `.env.local` file in the project root:
```
GA_KEY=your_google_analytics_4_measurement_id
```

### Development
```bash
npm run dev
# Runs Next.js dev server with hot reload
# PWA is disabled in development mode
# Accessible at http://localhost:3000
```

### Lint
```bash
npm run lint
# Runs ESLint with Next.js configuration
```

### Build
```bash
npm run build
# Creates optimized production build
# Generates service worker at public/sw.js
# Outputs static pages for serverless deployment
```

### Production
```bash
npm start
# Runs production server (after npm run build)
```

## Architecture

### Application Structure

The site is a single-page application (SPA) with all content on the home page ([pages/index.js](pages/index.js)):
- **Header** - Navigation and hero section
- **About** - Personal introduction
- **Skills** - Technical skills display
- **Portfolio** - Image gallery showcase (using react-image-gallery)
- **Footer** - Contact and social links

### Technology Stack

- **Next.js 16.0.10** - React framework with Pages Router and Turbopack
- **React 19.2.3** - UI library with latest features
- **Sass (Dart Sass 1.69.7)** - CSS preprocessing
- **@ducanh2912/next-pwa** - Modern PWA support
- **react-ga4** - Google Analytics 4 integration
- **react-image-gallery** - Portfolio image carousel
- **ESLint 9** - Code linting with Next.js configuration

### Key Files

- **[pages/_app.js](pages/_app.js)** - App wrapper that imports global SCSS and react-image-gallery CSS
- **[pages/index.js](pages/index.js)** - Main page component, initializes Google Analytics, assembles all sections
- **[pages/_document.js](pages/_document.js)** - Custom document for HTML structure and metadata
- **[pages/404.js](pages/404.js)** - Custom 404 error page
- **[next.config.js](next.config.js)** - Next.js configuration with PWA settings:
  - PWA configuration with workbox
  - Service worker with NetworkFirst caching strategy
  - React strict mode enabled
  - Turbopack configuration (Next.js 16+ default bundler)
  - Environment variables exposed to client

### Component Organization

Components are self-contained in folders under [components/](components/):
- Each component folder contains the component JS file and typically colocated data files
- Example: [components/Portfolio/](components/Portfolio/) contains [Portfolio.js](components/Portfolio/Portfolio.js) and [portfolioItems.js](components/Portfolio/portfolioItems.js)
- [components/Skills/](components/Skills/) contains [Skills.js](components/Skills/Skills.js) and [skills_data.js](components/Skills/skills_data.js)

### Styling System

Uses SCSS with a structured architecture pattern in [assets/sass/](assets/sass/):
- **abstracts/** - Variables, mixins, functions (reusable SCSS code)
- **base/** - Base styles, typography, animations, utilities
- **components/** - Component-specific styles
- **layout/** - Layout patterns (header, footer, navigation, grid)
- **pages/** - Page-specific styles
- **[main.scss](assets/sass/main.scss)** - Entry point importing all partials in specific order

Import order in main.scss matters: abstracts → base → components → layout → pages

**Note:** The SCSS uses deprecated `@import` syntax (should migrate to `@use` in future) and `@elseif` (should use `@else if`). These currently produce warnings but work fine.

### Image Handling

All images are stored in [public/images/](public/images/) and referenced with absolute paths:
- Use `/images/filename.jpg` in components
- Images are automatically optimized by Next.js
- react-image-gallery images use direct paths (not next/image component for compatibility)

### Environment Variables

Environment variables are handled natively by Next.js:
1. Create `.env.local` file in project root (not committed to git)
2. Add variables to [next.config.js](next.config.js) `env` object to expose to client-side
3. Currently configured: `GA_KEY` for Google Analytics 4

### Service Worker & PWA

Configured via @ducanh2912/next-pwa in [next.config.js](next.config.js):
- Service worker generated at `public/sw.js`
- Uses NetworkFirst strategy with 15s timeout
- Caches HTTPS calls for up to 1 month (max 150 entries)
- PWA disabled in development mode
- Enabled in production builds automatically

### Analytics

Google Analytics 4 integration via react-ga4:
- Initialized in [pages/index.js](pages/index.js) on mount
- Utility functions in [utils/googleAnalytics.js](utils/googleAnalytics.js)
- Requires `GA_KEY` environment variable (GA4 Measurement ID format: G-XXXXXXXXXX)

### Static Assets

- **Images**: [public/images/](public/images/) - Served from /images/ path
- **Public files**: [public/](public/) - manifest.json, favicon.png, apple-icon.png

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/phara0hcom) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
