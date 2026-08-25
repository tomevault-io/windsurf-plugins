---
trigger: always_on
description: Starter kit for building websites with **Astro 6** and **WordPress** as a headless CMS using **GraphQL**. Supports **i18n** with Spanish (default) and English via Polylang.
---

# CLAUDE.md - Project Guidelines

## Project Overview

Starter kit for building websites with **Astro 6** and **WordPress** as a headless CMS using **GraphQL**. Supports **i18n** with Spanish (default) and English via Polylang.

## Tech Stack

- **Framework**: Astro 6 (SSR mode, `output: "server"`)
- **Styling**: Tailwind CSS 4 (via `@tailwindcss/vite` plugin)
- **Deployment**: Vercel adapter (`@astrojs/vercel`) by default
- **CMS**: WordPress + WPGraphQL + Polylang
- **Package Manager**: pnpm (v10.33.2) - always use `pnpm`, never npm/yarn
- **Language**: TypeScript (strict mode)
- **Linting**: ESLint + Prettier

## Commands

```bash
pnpm dev          # Start dev server
pnpm build        # Production build
pnpm preview      # Preview production build (deploy runs on Vercel)
pnpm lint         # Run ESLint
pnpm lint:fix     # Fix ESLint issues
pnpm format       # Format with Prettier
pnpm format:check # Check formatting
pnpm type-check   # Run astro check (TypeScript)
```

## Project Structure

```
src/
  components/       # Reusable UI components (.astro)
    BlogCard.astro      # Blog post card for listings
    Breadcrumb.astro    # Breadcrumb navigation
    ImagePlaceholder.astro  # Fallback SVG when no image available
    PageHeader.astro    # Dark band: eyebrow + title + subtitle
    PageBody.astro      # WordPress page body (featured image + sanitized content)
    Lightbox.astro      # Image/YouTube modal (astro-icon); one per page, data-* triggers
    HeadlessForm.astro  # Renders a WPForms form headlessly (needs SW - WPForms GraphQL)
  i18n/             # i18n utilities (I18nUtils.ts)
  lib/              # Data layer & utilities
    graphql.ts      # GraphQL client (timeout + retry; optional {lang} for Polylang)
    blog.ts         # Blog post queries (WordPress native posts + Polylang)
    pages.ts        # WordPress Pages by slug (extend with your plugin's fields)
    menu.ts         # WordPress menus by location, Polylang-aware, normalized URLs
    siteSettings.ts # Site title/logo + ACF Options (contact, socials); cached per lang
    forms.ts        # WPForms schema via GraphQL (SW - WPForms GraphQL plugin)
    sitemap.ts      # Dynamic sitemap generation (queries WP for all slugs)
    url.ts          # URL builder helpers (buildUrl, isCurrentPage)
    utils.ts        # HTML/SVG sanitization (sanitize-html)
  locales/          # Translation strings, locale config, pageSlugs (Locales.ts)
  pages/            # Astro file-based routing
    index.astro     # Root redirect (302 + Vary: Accept-Language)
    404.astro       # Self-contained error page (no Layout dependency)
    500.astro       # Self-contained error page (no Layout dependency)
    sitemap-index.xml.ts  # Sitemap index endpoint
    sitemap-0.xml.ts      # Main sitemap with all URLs + hreflang
    [lang]/               # Locale-prefixed dynamic routes
      index.astro              # Home page
      [page].astro             # Generic WordPress page (resolved via pageSlugs)
      blog/index.astro         # Blog listing
      blog/[slug].astro        # Blog detail
      blog/feed.ts             # RSS feed (per language)
  theme/
    layouts/Layout.astro  # Main HTML layout (SEO, hreflang, OG/Twitter, JSON-LD, RSS)
    views/                # Page-level view components (HomeView, PageView, Blog*)
    styles/
      global.css          # Tailwind import, theme styles
      wordpress-content.css  # Styles for WordPress HTML content (.wp-content)
  types/
    blog.ts         # Post types and GraphQL responses
```

## Architecture Patterns

### Data Flow

Pages (`src/pages/`) fetch data and pass it to Views (`src/theme/views/`). Views compose Layout + Components. Components are pure presentational.

```
Page (data fetching) -> View (composition) -> Layout + Components
```

### GraphQL Client (`src/lib/graphql.ts`)

- Central `graphqlQuery<T>()` function with 10s timeout and AbortController
- All data modules (blog, sitemap) use this client
- `normalizeLanguageFilter()` converts locale to uppercase for WPGraphQL Polylang enum
- Error handling: each module catches errors, logs with `console.error`, returns empty/null fallback
- Uses `astro:env/server` for type-safe env var access

### i18n System

- **Locales.ts**: Single source of truth for all translation strings, locale config
- Both compile-time (`satisfies`) and runtime validation ensure ES/EN keys stay in sync
- **I18nUtils.ts**: `getLangFromUrl()`, `useTranslations()`, `getOtherLang()`, re-exports from `url.ts`
- All routes are prefixed with locale: `/{lang}/`, `/{lang}/blog/`, etc.
- `trailingSlash: "always"` - all URLs end with `/`

### URL Building

- Always use `buildUrl(lang, section?, slug?)` from `src/lib/url.ts`
- Sections type: `"blog"` (extend as needed)
- Slugs are URI-encoded via `encodeURIComponent`

### Security

- All WordPress HTML content is sanitized via `sanitize-html` before rendering
- Three sanitization functions: `stripHtml()`, `sanitizeHtml()`, `sanitizeSvg()`
- SVG sanitization preserves case sensitivity (`lowerCaseTags: false`)
- Environment variables use Astro's `envField` with `context: "server"` and `access: "secret"`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dazza-dev/astro-starter-kit](https://github.com/dazza-dev/astro-starter-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
