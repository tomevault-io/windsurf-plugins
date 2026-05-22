---
trigger: always_on
description: enableSpotlight={true}
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is Matthew Coleman's personal website built with Next.js 16 (App Router) as a static site. It features:

- **Blog** - Notion-powered blog with markdown rendering
- **Resume** - Professional resume/CV from Notion
- **Resource Library** - Curated links to websites and resources from Notion

The site is deployed to GitHub Pages at `https://mncoleman.github.io/mncoleman/`.

## Development Commands

```bash
npm run dev      # Start development server at http://localhost:3000
npm run build    # Build static site (outputs to out/)
npm run start    # Start production server (for testing build)
npm run lint     # Run ESLint
```

### Environment Setup

Create `.env.local` in the root directory with:

```env
# Notion integration token (starts with "ntn_")
NOTION_TOKEN=ntn_your_integration_token_here

# Blog database ID
NOTION_DATABASE_ID=your_blog_database_id_32_characters

# Resources database ID (separate database)
NOTION_RESOURCES_DATABASE_ID=your_resources_database_id_32_chars

# Resume page ID
NOTION_RESUME_PAGE_ID=your_resume_page_id_32_characters
```

**Important**: Notion tokens start with `ntn_`, not `secret_`. The code validates for placeholder values to gracefully fall back to sample data during development.

## Architecture

### Next.js Configuration

- **Output**: Static export (`output: 'export'`)
- **Base Path**: `/mncoleman` in production (GitHub Pages subpath)
- **Images**: Unoptimized (required for static export)
- **Trailing Slash**: Enabled for better compatibility

### Content Management Architecture

Uses a **two-layer adapter pattern** for all Notion content:

1. **lib/notion.ts** - Direct Notion API integration for blog posts
2. **lib/blog.ts** - Thin adapter for blog data
3. **lib/resources.ts** - Adapter for Resources database
4. **lib/resume.ts** - Adapter for Resume page

#### Credential Validation Pattern

All Notion data-fetching functions validate credentials BEFORE calling `getNotionClient()`:

```typescript
export async function getPublishedPosts(): Promise<NotionPost[]> {
    const databaseId = getDatabaseId();
    const token = process.env.NOTION_TOKEN;

    // Validate credentials before attempting connection
    if (!databaseId || !token || token === 'ntn_your_integration_token_here') {
        console.warn('Returning sample data because Notion credentials are not configured');
        return [/* sample data */];
    }

    try {
        const notion = getNotionClient();
        // ... API call
    } catch (error) {
        console.error('Error fetching posts:', error);
        return [/* sample data */];
    }
}
```

This pattern enables graceful degradation with sample data during development.

### Notion Data Sources

#### Blog Database

- Database ID: `your_blog_database_id_32_characters`
- Properties: Title, Slug, Date, Tags, Published, Featured, Excerpt, Author

#### Resources Database (separate database)

- Database ID: `your_resources_database_id_32_chars`
- Properties: Name, URL, Category, Description, Published

#### Resume Page

- Page ID: `your_resume_page_id_32_characters`
- Content: Markdown blocks to be converted

> **Important**: These are three separate Notion data sources. Blog and Resources are separate databases (not the same database with different views).

### Component Organization

- **components/ui/** - shadcn/ui + ReactBits components
  - `dark-veil.tsx` - WebGL background animation using OGL library
- **components/** - Custom components
- **app/** - Next.js App Router pages
- **worker/** - Cloudflare Worker for Admin Authentication & API

### Styling System

- **Tailwind CSS** with CSS variables for theming
- **Dark mode**: `next-themes` with class-based toggle
- **Component library**: shadcn/ui (New York style) + ReactBits registry
- **Frosted Glass Effect**: `bg-background/40 backdrop-blur-xl` for semi-transparent cards over Dark Veil

### Dark Veil Background Component

The home page uses a WebGL-based animated background (Dark Veil from ReactBits):

#### Key Implementation Details

- Canvas must use `position: fixed` with explicit `100vw/100vh` sizing
- Set `zIndex: -1` to keep background behind content
- Use `window.innerWidth` and `window.innerHeight` for resize calculations (not parent dimensions)
- Add `overflow-x: hidden` to html/body in globals.css to prevent cutoff
- Render directly without wrapper divs to avoid positioning conflicts
- The `resolutionScale` prop only affects render quality, not visual size

#### Example Usage

```typescript
<DarkVeil hueShift={40} speed={0.5} resolutionScale={0.8} />
```

#### Bento Cards Over Dark Veil

Use frosted glass transparency to show background colors:

```typescript
className="bg-background/40 backdrop-blur-xl border border-border/30"
```

---

## IMPLEMENTATION PLANS

The following plans should be executed by an AI model (Claude Sonnet or similar) to update this site.

---

## PLAN 1: Health Check & Optimization

### Findings

#### Current State Assessment

- File structure is clean and minimal
- Dependencies are appropriate for the project
- No major bloat or unnecessary files

#### Issues Identified


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mncoleman/mncoleman](https://github.com/mncoleman/mncoleman) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
