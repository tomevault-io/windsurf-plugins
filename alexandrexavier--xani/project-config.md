---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Xani is a personal blog and portfolio website built with Astro 5, featuring type-safe content collections, multi-language support (pt/en), and dynamic OG image generation. The repo/project is "Xani" but the public-facing site title (`SITE.title` in `src/config.ts`) is **"JAAX"** — keep this distinction in mind when editing copy, OG output, or schema markup.

See **`CONTEXT.md`** for project-specific terminology (Lab dropdown, Lab group, `/link` bookmarks). Terms defined there have a specific meaning in this codebase — prefer them over generic synonyms.

## Build & Development Commands

```bash
npm run dev          # Start dev server at localhost:4321
npm run build        # astro check (types) + astro build
npm run preview      # Preview production build locally
npm run lint         # ESLint check
npm run format       # Prettier format all files
npm run format:check # Check formatting without modifying
npm run sync         # Regenerate Astro TypeScript types
npm run cz           # Commitizen conventional-commit prompt
```

Note: `@divriots/jampack` is installed as a devDependency but is not wired into any npm script. If optimization is needed, run `npx jampack ./dist` after `npm run build`.

No test suite is configured.

## Architecture

### Content Collections (src/content/)
Type-safe markdown content with Zod schemas defined in `src/content/config.ts`:
- **blog/** - Blog posts: title, pubDatetime, modDatetime?, description, tags, language (pt/en, default en), draft, featured, ogImage (enforced ≥ 1200×630)
- **estudo/** - Study materials: title, category, tags (default `["direito"]`), language (default pt), draft
- **code/** - Code-focused notes: same shape as `estudo` with default tag `["code"]`
- **vela/** - Sailing notes (the Vela theme; see CONTEXT.md): same shape as `estudo` with default tag `["vela"]`, served at `/vela`

The registered collections are `{ blog, estudo, code, vela }` in `src/content/config.ts`. Note the directory is **blog/** but its public route is `/posts` (the listing layouts read the `blog` collection).

When adding a new collection, update `src/content/config.ts`, add a matching route under `src/pages/`, and run `npm run sync` to regenerate types before using the new schema.

### Key Directories
- **src/components/** - Astro (static) and React (interactive) components. Two notable subdirectories:
  - **schema/** - JSON-LD structured-data components (`BlogPostingSchema`, `BreadcrumbSchema`, `PersonWebsiteSchema`). Inject these in layouts/pages to emit SEO schema; do not hand-roll `<script type="application/ld+json">` elsewhere.
  - **pdf/** - React island for the `/pdf` page (`PDFConverterIsland` + sub-components and `hooks/`). Keep new PDF UI code here, not at the top level.
- **src/layouts/** - Page templates (Layout.astro is the main HTML wrapper; PostDetails, Posts, TagPosts, AboutLayout, Main)
- **src/pages/** - File-based routing. Top-level routes: `/` (index), `/about`, `/search`, `/404`, plus content sections `/posts` (the `blog` collection), `/estudo`, `/code`, `/vela`, `/tags`, and standalone features `/link`, `/pdf`, `/tempo`, `/gpg`. Endpoints: `og.png.ts`, `robots.txt.ts`, `rss.xml.ts`
- **src/utils/** - Post/episode filtering, sorting, pagination, tag aggregation, OG image generation (`og-templates/` holds JSX templates consumed by Satori). `pdf/` holds helpers used by the `/pdf` page and its worker.
- **src/config.ts** - `SITE` (website, author, postPerPage, scheduledPostMargin), `LOCALE`, `LOGO_IMAGE`, `SOCIALS`
- **src/types.ts** - Shared `Site` / `SocialObjects` types consumed by config
- **src/constants/** - Static data tables that drive UI without a content collection:
  - `links.ts` — bookmarks shown on the `/link` page (typed by `LinkItem`/`LinkCategory` in `src/types.ts`).
  - `labMenu.ts` — entries for the Lab dropdown in `Header.astro` (see CONTEXT.md for the Lab link vs. Lab group distinction; groups are one level deep only). Edit this file to add/reorder Lab menu items rather than touching the header markup.
- **src/workers/** - Vite ES-module workers (format set to `"es"` in `astro.config.ts`). `pdf-processor.worker.ts` offloads PDF parsing from the main thread for the `/pdf` page.
- **src/scripts/** - Client-side scripts (e.g. `tempo-chart.ts` drives the `/tempo` visualization)
- **src/helpers/** - Static JSON data: `themes.json`, `languagesList.json`, `podcastMainCategories.json`. No path alias exists for this directory; import with a relative path or use `public/` if needed client-side.
- **src/styles/base.css** - Tailwind directives and CSS custom properties for theming
- **public/** - Static assets served as-is (favicon, `toggle-theme.js`, `robots.txt` fallback)

### Standalone Lab Pages
- **`/link`** - Curated bookmarks driven entirely by `src/constants/links.ts`. Add a new link by appending a `LinkItem` to that file using the `LinkCategory` enum from `src/types.ts`. No content collection involved.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AlexandreXavier/xani](https://github.com/AlexandreXavier/xani) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
