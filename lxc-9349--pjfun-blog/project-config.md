---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**pjfun-blog** is a modern, fully static personal blog built with Vue 3 + Vite. It is a zero-backend SPA that generates all content at build time from Markdown/HTML/TXT/PDF/Word/Excel files placed in `public/content/`. The project serves its own documentation at [pjfun.top](https://pjfun.top).

## Key Design Decisions

1. **Content-driven**: All articles live as files in `public/content/`. The build plugin (`vite-plugin-gen-nav.ts`) scans them, parses frontmatter (YAML/gray-matter), and generates `nav.json` + `tree.json`.
2. **CDN-first production builds**: Dependencies (Vue, vue-router, highlight.js, pdfjs-dist, mammoth) are externalized and loaded from multiple CDN sources with automatic fallback (`package/build_cdn.ts` + `package/vite-plugin-cdn.ts`).
3. **Virtual module routing**: `vite-plugin-pages` auto-generates page routes from `src/pages/`. Article detail pages use a catch-all route (`:pathMatch(.*)*`) that resolves content at runtime by fetching `nav.json`.
4. **Client-side content fetching**: Articles are fetched as raw text from the `public/content/` directory at runtime, then rendered through `marked` (Markdown) or `mammoth` (Word) or `pdfjs-dist` (PDF).

## UI/UX Features (Current State)

- **Sticky header** with shrink-on-scroll effect (all pages)
- **Page transitions**: fade + translateY between routes (App.vue)
- **Mobile bottom nav** with 4 items (Home/Archive/Favorites/Search), safe-area-aware
- **Dark/light theme** with smooth CSS transition (0.3s)
- **Lenis smooth scroll** used throughout, including anchor navigation
- **Skeleton screens** matching card layout (2-column)
- **Card design**: 16:10 cover → title → excerpt (line-clamp-2) → tags + date + reading time + arrow in single row
- **Sticky badge**: gradient rose→amber pill
- **Right-click context menu** on cards (open in new tab / copy link / toggle favorite)
- **Ctrl+K search modal** with keyboard navigation (↑↓→), search cache
- **Reading progress bar** (top, gradient), auto-saves position
- **Image lightbox** with zoom
- **Code blocks**: language label top-left + copy button top-right, hover reveal
- **Font size toggle**: S/M/L
- **Share modal**: 8 platforms + copy link
- **Table of contents**: scroll-spy with IntersectionObserver, mobile drawer
- **Prev/Next article navigation** + keyboard ← → + mobile touch swipe
- **Tags**: horizontal scrollable pill row
- **Archive page**: timeline design (year badge + vertical line + month dots)
- **Favorites page**: mobile drawer sidebar

## Architecture (Build Pipeline)

```
vite-plugin-gen-nav.ts ──→ nav.json / tree.json
vite-plugin-auto-password-hash.ts ──→ SHA256 hash
vite-plugin-external + vite-plugin-cdn ──→ CDN injection
vite-plugin-rss.ts ──→ rss.xml / atom.xml / feed.json
vite-plugin-sitemap.ts ──→ sitemap.xml
vite-plugin-pwa ──→ Service Worker + manifest
vite-plugin-image-optimizer ──→ compressed assets
vite-plugin-html + EJS ──→ index.html with CDN vars
```

## Directory Structure

```
/
├── index.html                    # SPA entry (EJS template with CDN vars)
├── vite.config.ts                # Vite config (plugins, build, CDN, PWA, mime types)
├── uno.config.ts                 # UnoCSS config: keyframes, shortcuts, rules
├── vite-plugin-gen-nav.ts        # Content scan → nav/tree JSON + virtual routes
├── vite-plugin-auto-password-hash.ts  # SHA256 password hash at build time
├── package/
│   ├── build_cdn.ts              # CDN source config & multi-CDN fallback
│   ├── vite-plugin-cdn.ts        # Replace local JS with CDN scripts
│   ├── vite-plugin-rss.ts        # RSS feed generation
│   └── vite-plugin-sitemap.ts    # Sitemap XML generation
├── scripts/
│   ├── generate-encryption-key.cjs  # CLI password hash generator
│   └── worker.js                 # Web worker script
├── src/
│   ├── main.ts                   # App bootstrap: Lenis, theme, i18n, router, PWA, password guard
│   ├── App.vue                   # Root: router-view + page transition + bottom nav + update notification
│   ├── constants/
│   │   └── index.ts              # SITE_CONFIG, GISCUS_CONFIG, HOT_TAGS, I18N_CONFIG
│   ├── styles/
│   │   └── global.css            # Global styles (scrollbar hide, toc-scrollbar, ease-out-back)
│   ├── pages/
│   │   ├── index.vue             # Home: sticky header, card grid, tag filter, right-click menu, Lenis
│   │   ├── archive.vue           # Timeline: year/month grouped, sticky header
│   │   ├── favorites.vue         # Favorites: card grid, mobile drawer, sticky header
│   │   └── articleDetail.vue     # Article: cover, code blocks, TOC, share, print, keyboard nav, swipe
│   ├── components/
│   │   ├── NavTree.vue           # Recursive nav tree with localStorage persistence
│   │   ├── DocumentViewer.vue    # PDF/Word/Excel viewer wrapper
│   │   ├── GiscusComment.vue     # Giscus comment system
│   │   ├── PasswordProtection.vue  # Password-protected access form
│   │   ├── RecentArticles.vue    # Recent articles sidebar
│   │   ├── Footer.vue            # Footer with Buzuanzi stats
│   │   ├── preview/
│   │   │   ├── PdfToHtml.vue     # PDF renderer (canvas-based, pdfjs-dist)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LXC-9349/pjfun-blog](https://github.com/LXC-9349/pjfun-blog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
