---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Core Development

```bash
# Start development server (localhost:4321)
bun run dev

# Build for production (includes TypeScript checking)
bun run build

# Preview production build
bun run preview

# TypeScript checking only
bunx astro check

# Generate content collection types
bunx astro sync
```

### Package Manager

This project uses **bun** as the preferred package manager (evidenced by `bun.lockb`). Always use `bun` commands when managing dependencies.

## Architecture Overview

### Core Stack

- **Astro 5.14+** with static site generation (`output: 'static'`)
- **React 19** components with selective hydration via `client:load`
- **shadcn/ui** complete component library (50+ components pre-installed)
- **Tailwind CSS v4** with modern CSS-based configuration and dark mode
- **TypeScript 5.9+** in strict mode with path aliases
- **Nanostores** for lightweight state management
- **Recharts 2.15+** for data visualization
- **@astrojs/sitemap** for automatic sitemap generation
- **@astrojs/rss** for RSS feed generation

### Key Architectural Patterns

**Islands Architecture**: This project follows Astro's islands pattern where:

- Pages are `.astro` files that render statically by default
- React components need `client:load` directive for interactivity
- Most UI is static for optimal performance

**Content Collections**: Blog content uses Astro's type-safe content collections:

- Content schema defined in `src/content/config.ts` with Zod validation
- Blog posts in `src/content/blog/` with frontmatter
- Dynamic routing via `[...slug].astro` with static generation

**Theme System**: Advanced dark/light mode implementation with Tailwind v4:

- CSS custom properties in `src/styles/global.css` using HSL color format
- `@variant dark (.dark &)` for dark mode support
- `ThemeInit.astro` component prevents FOUC (Flash of Unstyled Content)
- localStorage persistence with server-side rendering support
- Dark class applied to `<html>` element via JavaScript

### Path Aliases

```typescript
"@/*": ["src/*"]
"@components/*": ["src/components/*"]
"@layouts/*": ["src/layouts/*"]
"@lib/*": ["src/lib/*"]
"@stores/*": ["src/stores/*"]
"@content/*": ["src/content/*"]
"@hooks/*": ["src/hooks/*"]
"@types/*": ["src/types/*"]
"@config/*": ["src/config/*"]
```

### Component Architecture

**shadcn/ui Integration**: Complete component library in `src/components/ui/`

- Components follow shadcn conventions with Radix UI primitives
- Use `className` prop for styling (not `class`)
- All components are React-based and need `client:load` for interactivity

**Custom Components**:

- `Sidebar.tsx`: Expandable navigation with hover states and active route detection
- `ModeToggle.tsx`: Dark/light mode switcher
- `Chart.tsx`: Recharts integration for data visualization
- `ThemeInit.astro`: Theme initialization without JavaScript flash
- `BlogSearch.tsx`: Real-time blog post search with filtering
- `TableOfContents.tsx`: Auto-generated ToC with IntersectionObserver tracking
- `ShareButtons.tsx`: Native Web Share API + social media buttons
- `ErrorBoundary.tsx`: React error boundary with alert UI

### Blog System Features

**Multi-View Blog Interface**:

- List view and grid view (2/3/4 columns) controlled by URL parameters
- View mode switching: `/blog?view=grid&columns=3`
- Real-time search filtering by title and description
- Responsive image handling with lazy loading
- Date formatting with `Intl.DateTimeFormat`

**Content Structure**:

```typescript
// Blog schema (src/content/config.ts)
const BlogSchema = z.object({
  title: z.string(),
  description: z.string(),
  date: z.date(),
  draft: z.boolean().optional(),
  image: z.string().optional(),
  author: z.string().default('ONE'),
  tags: z.array(z.string()).default([]),
  category: z.enum(['tutorial', 'news', 'guide', 'review', 'article']).default('article'),
  readingTime: z.number().optional(),
  featured: z.boolean().default(false),
});
```

**Blog Features**:

- **Real-time Search**: BlogSearch component with instant filtering
- **Table of Contents**: Auto-generated from markdown headings with active tracking
- **Social Sharing**: Native Web Share API + Twitter, Facebook, LinkedIn buttons
- **Tags & Categories**: Rich metadata for content organization
- **RSS Feed**: Auto-generated at `/rss.xml` with all blog posts
- **Sitemap**: Auto-generated with `@astrojs/sitemap`

### State Management

**Nanostores Pattern**: Lightweight reactive state

- Layout state in `src/stores/layout.ts`
- localStorage persistence
- Cross-component reactivity with `@nanostores/react`

### Styling Architecture

**Tailwind CSS v4 Configuration**: Modern CSS-based configuration

- No `tailwind.config.mjs` file - configuration in CSS using `@theme` blocks
- Uses `@tailwindcss/vite` plugin in `astro.config.mjs`
- CSS imports via `@import "tailwindcss"`
- `@plugin "@tailwindcss/typography"` for typography support
- Dark mode via `@variant dark (.dark &)` selector

**Global Styles**: `src/styles/global.css` defines:

- `@theme` blocks with HSL color values (e.g., `--color-background: 0 0% 100%`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [one-ie/astro-shadcn](https://github.com/one-ie/astro-shadcn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
