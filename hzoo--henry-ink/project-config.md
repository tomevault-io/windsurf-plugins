---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is **henry.ink** - a social annotation layer for the web. What if every webpage had marginalia for notes? Originally developed as a browser extension for the Community Archive hackathon, it has evolved into a web-first platform at henry.ink that works without any installation. The project maintains both the web application and the original browser extension (Chrome & Firefox) for integrated browsing experiences.

## Conceptual Overview

This project takes a "reverse Hypothesis" approach to web annotations. Instead of overlaying annotations on existing webpages (like traditional annotation tools), we recreate the webpage content within our own environment. This gives us full control over the UI, enabling seamless integration of comments with the original content - such as automatically highlighting annotated text. The goal is to create a unified reading experience where blog posts and their associated discussions from Bluesky, Are.na, and other sources feel naturally integrated rather than bolted on.

## Technology Stack

- **Framework**: Preact (React-like, optimized for performance)
- **Build Tool**: WXT (Modern browser extension framework)
- **Language**: TypeScript with strict configuration
- **Styling**: Tailwind CSS v4 with Vite plugin
- **Package Manager**: Bun
- **State Management**: Preact Signals
- **Data Fetching**: TanStack Query with persistence
- **Backend**: Bluesky/AT Protocol via `@atcute` libraries
- **Linting**: Biome.js (replaces ESLint/Prettier)

## Development Commands

```bash
# Development
bun run ink          # Henry.ink development server (primary)
bun run dev          # AT Protocol extension development (Chrome)
bun run demo         # Web app demo version (annotation-demo site)

# Building
bun run build        # Build AT Protocol extension
bun run build:demo   # Build annotation demo
bun run build:ink    # Build henry.ink

# Extension Development (in extension/ directory)
cd extension
bun dev              # Chrome extension development
bun dev:ff           # Firefox extension development
bun build            # Build Chrome extension
bun build:ff         # Build Firefox extension
bun zip              # Package for Chrome Web Store
bun zip:ff           # Package for Firefox Add-ons

# Code Quality
bun run compile      # TypeScript type checking
bun run lint         # Biome linting and formatting (if configured)

# Deployment
bun run deploy:demo      # Deploy demo to Cloudflare Pages
bun run deploy:ink       # Deploy henry.ink to Cloudflare Pages
bun run deploy:bsky_worker # Deploy Bluesky proxy worker
bun run deploy:jina_worker # Deploy Jina proxy worker
bun run release          # Release new version
```

## Architecture

### Multi-Platform Structure
- **henry.ink**: Primary web application at `/henry-ink/` - renders any URL with integrated social discussions
- **AT Protocol Extension**: Browser extension workspace at `/extension/` - integrated browsing with AT Protocol data
- **Demo Sites**:
  - `demo/` - Annotation demo website
  - `archive-service/` - Secure web page archiving service
- **Shared Components**: `/src/components/` used across all platforms

### henry.ink Web Application

The **henry.ink** website is the primary platform - a full-featured web application that provides annotation capabilities for any URL without requiring downloads or installation. It implements the "reverse Hypothesis" approach by recreating webpage content in a controlled environment with integrated social discussions.

#### Core Features
- **Universal URL Support**: Renders any webpage by appending URL to `henry.ink/[url]`
- **Reader Mode**: Clean, readable formatting of web content with markdown rendering
- **Social Annotations**: Integrated Bluesky discussions displayed in sidebar
- **Text Selection & Annotation**: Full annotation capabilities with text highlighting
- **Profile Pages**: View individual users' annotations across different websites
- **No Extension Required**: Works entirely in the browser without downloads

#### Routing Structure
- **`/`** - Landing page with usage instructions and bookmarklet
- **`/profile/:username`** - User profile showing their annotations across sites
- **`/:params*`** - Catch-all route for any URL (e.g., `/https://example.com/article`)
- **Query Parameters**: `?post=rkey` for auto-scrolling to specific discussions

#### Key Components
- **MarkdownSite**: Main content rendering with highlight integration
- **ProfilePage**: User annotation history and activity
- **HighlightController**: Text selection and annotation management
- **Sidebar**: Bluesky discussions and social interactions

#### Usage Patterns
1. **Direct URL**: Visit `henry.ink/https://example.com` for any webpage
2. **Bookmarklet**: One-click access from any page via bookmark bar
3. **Profile Navigation**: Browse user annotations via `/profile/username`
4. **Deep Linking**: Share specific discussions with `?post=` parameters

### State Management Pattern
- **Global state**: Preact Signals in `/src/lib/signals.ts`
- **Component state**: `useSignal()` hooks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hzoo/henry.ink](https://github.com/hzoo/henry.ink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
