---
trigger: always_on
description: vibe.j2team.org — A collaborative vibe coding project by J2TEAM Community with 210+ sub-apps. The homepage acts as a launcher linking to sub-apps, where each community member creates their own page. Deployed to Cloudflare Workers (static assets via `wrangler.json`).
---

# AGENTS.md

## Project Overview

vibe.j2team.org — A collaborative vibe coding project by J2TEAM Community with 210+ sub-apps. The homepage acts as a launcher linking to sub-apps, where each community member creates their own page. Deployed to Cloudflare Workers (static assets via `wrangler.json`).

## Tech Stack

- Vue 3.5 (Composition API with `<script setup>`)
- TypeScript (strict mode, `noUncheckedIndexedAccess: true`)
- Vite 7
- Tailwind CSS v4 (via `@tailwindcss/vite`)
- Vue Router 5
- Pinia 3
- @unhead/vue 2 (document head/meta management)
- @vueuse/core 14 — 200+ Vue composables (useMouse, useClipboard, useDark, useStorage, useIntersectionObserver, useLocalStorage, useMediaQuery, useWindowSize, etc.)
- @iconify/vue 5 — 200,000+ icons from 150+ icon sets via `<Icon icon="icon-set:icon-name" />` component
- html-to-image — capture DOM nodes as PNG/JPEG/SVG
- shiki 4 — syntax highlighting

## Setup & Build

```sh
pnpm install
pnpm dev          # Dev server
pnpm build        # Type-check + production build (+ OG pages + sitemap generation)
pnpm test:unit    # Unit tests with Vitest
pnpm lint         # Lint with oxlint + ESLint (with --fix)
pnpm lint:ci      # Lint without --fix (for CI)
pnpm format       # Format with oxfmt
pnpm create:page <slug>  # Scaffold a new page (interactive or with flags)
```

## Project Structure

```
src/
  main.ts                    # App entry (createPinia + createHead + router)
  App.vue                    # Root component (<RouterView /> + useHead for dynamic meta)
  assets/main.css            # Tailwind CSS v4 @theme tokens + custom animations
  router/index.ts            # Vue Router — auto-generates routes from pages-loader
  types/page.ts              # PageMeta & PageInfo interfaces
  data/
    pages-loader.ts          # Fetches pre-generated pages.json (built by Vite plugin)
    categories.ts            # Category definitions (game, tool, creative, fun, learn, health, finance, spiritual, connect, other)
    authors.ts               # Author aggregation from pages (leaderboard, author pages)
    homepage.ts              # Homepage content data (tech stack, rules, products)
    constants.ts             # Shared constants
  components/
    home/                    # Homepage section components (HeroSection, PagesGrid, RecentlyViewed, etc.)
    AppBreadcrumb.vue        # Standardized breadcrumb navigation for core pages
    AuthorAvatar.vue         # Author avatar (GitHub avatar via useGithubAvatar)
    BackToTop.vue
    CategoryFilter.vue       # Category filter UI for homepage
    EdgeToolbar.vue          # Slide-out toolbar on sub-pages (source link, bookmark, home, comments)
    ErrorBoundary.vue        # Error boundary wrapper
    FavoriteButton.vue       # Bookmark/favorite toggle button
    GiscusModal.vue          # Giscus comments modal (per-page discussions)
    PageCard.vue             # Page card component used in grids
  composables/
    useDraggable.ts          # Drag behavior composable
    useGithubAvatar.ts       # Resolves GitHub avatar URL from author name
    useSearchShortcut.ts     # Keyboard shortcut for search (Ctrl+K / Cmd+K)
  stores/
    useFavoritesStore.ts     # Bookmark/favorite state (localStorage via VueUse)
    usePagesStore.ts         # Pages registry (fetches pages.json, provides page list)
    useRecentlyViewedStore.ts # Tracks recently visited pages
  views/
    HomePage.vue             # Landing page / launcher
    ContentPolicy.vue        # Content policy page
    LeaderboardPage.vue      # Author leaderboard (/leaderboard)
    BookmarksPage.vue        # User's bookmarked pages (/bookmarks)
    AuthorPage.vue           # Author profile page (/author/:slug)
    NotFound.vue             # 404 page
    <app-name>/
      index.vue              # Each sub-page is a directory with index.vue
      meta.ts                # Page metadata — route auto-generated from folder name
```

## Auto-Routing System

Routes are auto-generated from a pre-built `public/data/pages.json` file:
- A Vite plugin (`scripts/generate-pages-json.mjs`) scans all `src/views/*/meta.ts` files and writes `public/data/pages.json` at build start and whenever a `meta.ts` file changes during dev
- Post-build scripts generate OG image pages (`scripts/generate-og-pages.mjs`) and sitemap with sub-sitemaps + robots.txt (`scripts/generate-sitemap.mjs`)
- `src/data/pages-loader.ts` fetches this JSON at runtime (bypasses Rollup bundling)
- Path is derived from folder name (e.g., `src/views/my-app/` → `/my-app`)
- Featured pages are hand-picked in `scripts/generate-pages-json.mjs` and pinned to top of homepage
- Pages with `hidden: true` in their `meta.ts` are excluded from the listing but their routes still work

## Design System

**IMPORTANT**: All UI work MUST follow the design system documented in `docs/DESIGN_SYSTEM.md`.

Key rules:
- Use the custom color tokens defined in `src/assets/main.css` (`@theme` block) — never use raw Tailwind grays or default colors
- DO NOT use purple, green-cyan gradients, or cold grays (`gray-950`, `gray-900`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [J2TEAM/vibe.j2team.org](https://github.com/J2TEAM/vibe.j2team.org) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
