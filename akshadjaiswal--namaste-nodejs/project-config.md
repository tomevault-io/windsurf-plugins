---
trigger: always_on
description: This file gives Claude (and any AI assistant) the context needed to work effectively on this Next.js documentation app.
---

# CLAUDE.md — Namaste Node.js Documentation App

This file gives Claude (and any AI assistant) the context needed to work effectively on this Next.js documentation app.

## What this app is

A statically generated documentation site built with Next.js 16. It reads Node.js learning notes from the parent repository and presents them as a navigable, readable documentation site. There are 35 chapters total across 3 seasons.

The app lives at: `Namaste-Nodejs/application/`
The parent repo root (content source) is: `Namaste-Nodejs/` (one level up via `process.cwd() + '/...'`)

## Content source — read this carefully

Content is NOT stored inside `application/`. It is read at build time from the parent repository:

| Season | Source | How parsed |
|--------|--------|------------|
| Season 1 (13 chapters) | `../Chapter 01 - Title/README.md` … `../Chapter 13 - Title/README.md` | Directory scan — each directory is one chapter page |
| Season 2 (19 chapters) | `../Chapter S2 01 Title/README.md` … `../Chapter S2 19 Title/README.md` | Same directory scan, `S2` prefix distinguishes season |
| Season 3 (3 chapters) | `../Chapter S3 01 Title/README.md` … `../Chapter S3 03 Title/README.md` | Same pattern, `S3` prefix |

The entire parsing logic lives in `lib/chapters.ts`. Do NOT move content files into `application/`.

## Directory naming convention

Chapter directories are inconsistent — some have dash separators, some don't. The parser handles both:

```
Pattern 1 (with dash):    Chapter 01 - Introduction to NodeJs
Pattern 2 (no dash):      Chapter 02 JS on the Server
Season 2 (no dash):       Chapter S2 01 Microservices vs Monolith - How to build a project
Season 3:                 Chapter S3 01 Lanching an AWS Instance and Deploying frontend
```

Primary regex: `/^Chapter\s+(S(\d)\s+)?(\d+)\s*[-–]\s*(.+)$/`
Fallback regex: `/^Chapter\s+(S(\d)\s+)?(\d+)\s+(.+)$/`

Both are handled in `parseDirName()` in `lib/chapters.ts`.

## Architecture

```
application/
├── app/
│   ├── layout.tsx              # Root layout: sidebar + header + children
│   ├── page.tsx                # Home: hero + season grids
│   ├── globals.css             # Tailwind base + .shiki styling + .author-link
│   ├── error.tsx               # Error boundary (client component)
│   ├── robots.ts               # robots.txt metadata
│   ├── sitemap.ts              # Dynamic sitemap from getAllChapters()
│   ├── opengraph-image.tsx     # OG image (site-level)
│   └── chapters/[slug]/
│       ├── page.tsx            # Chapter page with TOC + nav + ReadingProgress
│       ├── loading.tsx         # Skeleton loading state
│       ├── not-found.tsx       # 404 for bad slugs
│       └── opengraph-image.tsx # Per-chapter OG image (1200×630, Node.js green)
├── components/
│   ├── sidebar.tsx             # Server wrapper — calls getSeasons()
│   ├── sidebar-client.tsx      # Client: collapsible nav, mobile overlay, completion checkmarks, progress bar, reset button
│   ├── header.tsx              # Sticky header with GitHub stars + SearchTrigger + ShortcutsTrigger + ThemeToggle + attribution
│   ├── markdown-renderer.tsx   # Async RSC: Shiki dual-theme highlighted react-markdown + HeadingAnchor
│   ├── table-of-contents.tsx   # Client: sticky TOC with intersection observer, active heading, scroll % indicator
│   ├── chapter-nav.tsx         # Prev/Next chapter links with read time
│   ├── copy-button.tsx         # 'use client' — copy-to-clipboard for code blocks
│   ├── reading-progress.tsx    # 'use client' — amber scroll progress bar (chapter pages only)
│   ├── search-modal.tsx        # 'use client' — Fuse.js search modal (fetch + fuzzy search)
│   ├── search-trigger.tsx      # 'use client' — search icon button + `/` global shortcut
│   ├── bookmark-button.tsx     # 'use client' — bookmark toggle; also records last visited
│   ├── continue-reading.tsx    # 'use client' — sidebar continue reading / bookmark link
│   ├── theme-provider.tsx      # 'use client' — ThemeContext, localStorage sync, html.dark toggle
│   ├── theme-toggle.tsx        # 'use client' — Sun/Moon icon button in header
│   ├── heading-anchor.tsx      # 'use client' — copy-link-to-section icon on headings
│   ├── complete-button.tsx     # 'use client' — "Mark Complete" toggle; shows season-complete celebration banner
│   ├── chapter-completion-badge.tsx  # 'use client' — amber checkmark on home page cards
│   ├── shortcuts-modal.tsx     # 'use client' — keyboard shortcuts reference modal
│   ├── shortcuts-trigger.tsx   # 'use client' — '?' key listener + Keyboard icon button
│   ├── chapter-shortcuts.tsx   # 'use client' — chapter-page-scoped 'b' key → bookmark
│   ├── print-button.tsx        # 'use client' — Printer icon button; calls window.print()
│   └── scroll-to-top.tsx       # 'use client' — floating ArrowUp button, appears after 300px scroll
├── hooks/
│   └── use-bookmark.ts         # useLastVisited, useBookmark, useContinueReading, useCompletedChapters hooks
├── lib/
│   ├── chapters.ts             # ALL content parsing — the core of the app
│   ├── github.ts               # Fetches GitHub star count (cached 1h)
│   └── utils.ts                # cn() utility (clsx + tailwind-merge)
├── types/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [akshadjaiswal/Namaste-Nodejs](https://github.com/akshadjaiswal/Namaste-Nodejs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
