---
trigger: always_on
description: This file gives Claude (and any AI assistant) the context needed to work effectively on this Next.js documentation app.
---

# CLAUDE.md — Namaste JavaScript Documentation App

This file gives Claude (and any AI assistant) the context needed to work effectively on this Next.js documentation app.

## What this app is

A statically generated documentation site built with Next.js 16. It reads JavaScript learning notes from the parent repository and presents them as a navigable, readable documentation site — similar to how library docs are structured. There are 26 pages total across 3 seasons.

The app lives at: `Namaste-JavaScript/application/`
The parent repo root (content source) is: `Namaste-JavaScript/` (one level up via `process.cwd() + '/...'`)

## Content source — read this carefully

Content is NOT stored inside `application/`. It is read at build time from the parent repository:

| Season | Source | How parsed |
|--------|--------|------------|
| Season 1 (19 episodes) | `../Chapter 01 - Title/README.md` … `../Chapter 19 - Title/README.md` | Directory scan — each directory is one episode page |
| Season 2 (5 episodes) | `../Chapter S2 01 - Title/README.md` … `../Chapter S2 05 - Title/README.md` | Same directory scan, `S2` prefix distinguishes season |
| Concepts (2 topics) | `../Concepts/*/README.md` | Each subdirectory is one concept page |

The entire parsing logic lives in `lib/chapters.ts`. Do NOT move content files into `application/`.

## Directory naming convention

Chapter directories follow this exact pattern (matched by regex `^Chapter\s+(S(\d)\s+)?(\d+)\s*[-–]\s*(.+)$`):

```
Season 1:  Chapter 01 - Execution Context/README.md
           Chapter 02 - Execution and Call Stack/README.md
           ...
           Chapter 19 - map filter and reduce/README.md
Season 2:  Chapter S2 01 - Callback Hell/README.md
           ...
           Chapter S2 05 - this Keyword in JavaScript/README.md
Concepts:  Concepts/Debouncing/README.md
           Concepts/Throtling/README.md   (folder typo — slug is concepts-throtling)
```

## Concepts folder note

The Concepts folder `Throtling` has a one-`t` typo. The slug is `concepts-throtling` (stable, don't rename). The display title is normalized to "Throttling" via `CONCEPT_TITLE_MAP` in `lib/chapters.ts`.

## Architecture

```
application/
├── app/
│   ├── layout.tsx              # Root layout: ThemeProvider + sidebar + header + children + blocking script
│   ├── page.tsx                # Home: hero + season grids + ChapterCompletionBadge
│   ├── globals.css             # Tailwind base + dark mode + .shiki styling + .author-link
│   ├── error.tsx               # Error boundary (client component)
│   ├── robots.ts               # robots.txt metadata
│   ├── sitemap.ts              # Dynamic sitemap from getAllChapters()
│   ├── opengraph-image.tsx     # OG image (site-level, amber bar + title)
│   └── chapters/[slug]/
│       ├── page.tsx            # Episode/concept page with TOC + nav + ReadingProgress + CompleteButton + ChapterShortcuts
│       ├── loading.tsx         # Skeleton loading state
│       ├── not-found.tsx       # 404 for bad slugs
│       └── opengraph-image.tsx # Per-chapter OG image (1200×630, JS amber)
├── components/
│   ├── sidebar.tsx             # Server wrapper — calls getSeasons()
│   ├── sidebar-client.tsx      # Client: collapsible nav, mobile overlay, completion checkmarks, progress bar
│   ├── header.tsx              # Sticky header with GitHub stars + ShortcutsTrigger + ThemeToggle + SearchTrigger
│   ├── markdown-renderer.tsx   # Async RSC: Shiki dual-theme highlighted react-markdown + HeadingAnchor
│   ├── table-of-contents.tsx   # Client: sticky TOC with intersection observer
│   ├── chapter-nav.tsx         # Prev/Next episode links
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

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [akshadjaiswal/Namaste-JavaScript](https://github.com/akshadjaiswal/Namaste-JavaScript) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
