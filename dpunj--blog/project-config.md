---
trigger: always_on
description: Project context and instructions for AI assistants working on this codebase.
---

# CLAUDE.md

Project context and instructions for AI assistants working on this codebase.

## Project Overview

Personal blog for divesh.gg built with Astro, Preact, and Tailwind CSS v4. This is a content-focused site with two types of posts: "waves" (short posts) and "depths" (long-form content).

## Tech Stack

- **Framework**: Astro 4.x with Preact integration
- **Styling**: Tailwind CSS v4 (via @tailwindcss/vite)
- **Content**: Markdown with frontmatter, managed via Astro Content Collections
- **Type Safety**: TypeScript with Zod schema validation
- **Tooling**: Biome for linting and formatting
- **Package Manager**: Bun (preferred) or npm

## Project Structure

```
src/
├── components/
│   ├── library/     # Library explorer components (Preact)
│   └── ...          # Other UI components (.astro, .jsx, .tsx)
├── content/
│   ├── waves/       # Short-form posts (markdown)
│   └── depths/      # Long-form posts (markdown)
├── content.config.ts # Content collections schema
├── data/
│   └── tag-hierarchy.json  # Library tag organization
├── layouts/         # Page layout templates
├── pages/           # File-based routing
├── scripts/         # Utility functions (library.ts, music.ts, etc.)
└── styles/          # Global CSS
scripts/
├── db.ts            # SQLite schema and CRUD helpers (bun:sqlite)
└── sync.ts          # Unified CLI for syncing all data sources
data/
└── knowledge.db     # SQLite database (source of truth)
public/data/
└── library.json     # Exported from SQLite for Astro build
```

## Development Workflow

### Commands
- `bun dev` - Start dev server (localhost:4321)
- `bun build` - Production build to ./dist/
- `bun preview` - Preview production build
- `bun lint` - Check for issues
- `bun lint:fix` - Auto-fix linting issues
- `bun format` - Format all files
- `bun sync local` - Sync local sources (books, music) - fast
- `bun sync api` - Sync API sources (Readwise, Zotero) - slow, rate limited
- `bun sync --export-library` - Export library from SQLite to JSON for Astro

### Package Manager
- **Prefer Bun** for all package operations
- Fallback to npm if needed

## Code Style & Standards

### Formatting (Biome)
- **Indentation**: Tabs (not spaces)
- **Quotes**: Double quotes for JavaScript/TypeScript
- **CSS**: Tailwind directives enabled
- **Astro files**: Formatting and linting disabled (handled by Astro)

### File Conventions
- Components: PascalCase (e.g., `BlogPost.astro`, `Header.astro`)
- Use `.astro` for server components
- Use `.jsx`/`.tsx` for Preact interactive components
- Content files: kebab-case markdown (e.g., `how-videogames-shaped-me.md`)

## Content Management

### Blog Post Schema
All posts require frontmatter:

```yaml
---
title: string           # Required
author: string          # Required
description: string     # Required (for SEO/previews)
pubDate: Date          # Required (YYYY-MM-DD format)
tags: string[]         # Required
image?: string         # Optional image URL
---
```

### Content Types
- **waves/**: Short posts, updates, quick thoughts
- **depths/**: Long-form articles, essays, deep dives

### Adding Content
1. Create markdown file in `src/content/waves/` or `src/content/depths/`
2. Add required frontmatter
3. Schema validation enforced via `content.config.ts`

## Key Integrations

### Tailwind CSS v4
- Uses new Vite plugin (`@tailwindcss/vite`)
- Configuration via CSS (not tailwind.config.js)

### Preact
- For interactive components requiring client-side JS
- Lighter alternative to React
- Use `client:*` directives in Astro for hydration

### Library (Digital Garden)
A browsable archive of bookmarks (Readwise) and papers (Zotero) with hierarchical tag navigation.

**Data Flow:**
```
Readwise/Zotero APIs → sync.ts → SQLite sync_state/resources → --export-library → library.json → /library page
```

**Key Files:**
- `scripts/sync.ts` - Unified sync CLI with API fetchers and rate limiting
- `scripts/db.ts` - SQLite schema and CRUD helpers
- `src/scripts/library.ts` - Types, tag hierarchy builder, filters, sorting
- `src/data/tag-hierarchy.json` - Manual tag → hierarchy mapping (edit this!)
- `src/components/library/` - Preact components (LibraryExplorer, TagTree, ResourceList, etc.)
- `public/data/library.json` - Exported for Astro (regenerate with `bun sync --export-library`)

**Environment Variables** (`.env`):
```
READWISE_TOKEN=xxx
ZOTERO_API_KEY=xxx
ZOTERO_USER_ID=xxx
LETTERBOXD_USERNAME=xxx
RAWG_API_KEY=xxx
RAWG_USERNAME=xxx
TMDB_API_KEY=xxx
```

**Updating the Library:**
1. Run `bun sync api --export-library` to fetch latest resources and regenerate JSON
2. Use `bun sync readwise --full` or `bun sync zotero --full` only for full backfills
3. Edit `src/data/tag-hierarchy.json` to organize new tags
4. Uncategorized tags appear at the bottom of the tag tree

## Important Notes

- **SEO Component**: Uses TypeScript (.tsx) - maintain type safety
- **Navigation**: Check `Navigation.astro` for site structure
- **Git**: Main branch is `main`
- **Images**: Currently using placeholder images, update as needed
- **Domain**: divesh.gg (not yet registered - noted in README)

## Common Tasks

### Adding a New Post
1. Create `.md` file in appropriate content directory
2. Add complete frontmatter with all required fields

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dpunj/blog](https://github.com/dpunj/blog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-21 -->
