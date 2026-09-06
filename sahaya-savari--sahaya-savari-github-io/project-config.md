---
trigger: always_on
description: This document is the operating guide for AI agents working on this repository. Every agent must read it, keep it accurate, and follow it. All statements below are verifiable from the current codebase.
---

# Operating Manual for AI Coding Agents (AGENTS.md)

This document is the operating guide for AI agents working on this repository. Every agent must read it, keep it accurate, and follow it. All statements below are verifiable from the current codebase.

---

## Project Overview

- **Project name:** `sahaya-savari-blog` (package name). The site brands itself as the **Sahaya Savari Blog**.
- **Purpose:** A technical learning blog that publishes long-form tutorials on Python, Git & GitHub, programming fundamentals, and React/web development.
- **Tech stack:** React 18, TypeScript (strict), Vite 5, Tailwind CSS 3, Framer Motion, Lucide React, React Router DOM 6, and MDX (`@mdx-js/rollup` + `@mdx-js/react`).
- **Build tool:** Vite (`vite build`, preceded by `tsc -b` for type checking).
- **Package manager:** npm (a `package-lock.json` is committed).
- **Deployment method:** GitHub Pages via a GitHub Actions workflow (`.github/workflows/deploy.yml`), served on the custom domain `blog.sahayasavari.dev` (see `CNAME` and `public/CNAME`).
- **Repository structure:** A single-page React application. Blog content lives as MDX files under `content/`; built static assets are deployed from `dist/`.

---

## Architecture

- **Single-page app.** `src/main.tsx` mounts `<App />` inside `<BrowserRouter>` (from `react-router-dom`) and `<React.StrictMode>`. It also imports the global stylesheet and a Prism theme for code highlighting.
- **Route composition.** `src/App.tsx` renders a persistent `Navbar`, `Footer`, `ScrollToTop`, and an `AnimatePresence` wrapper around the routed `<main>`. Page components are code-split with `React.lazy` + `Suspense`.
- **Content pipeline.** MDX files in `content/` are compiled at build time by the `@mdx-js/rollup` plugin configured in `vite.config.ts`. `src/lib/data.ts` uses `import.meta.glob` to eagerly load both the compiled MDX modules and their raw text, then derives the `blogPosts` array and a `postComponents` slug→component map.
- **Static, client-side data.** There is no backend or data-fetching layer. Editorial data (categories, comments, testimonials, timeline, skills, achievements) is declared as typed constants in `src/lib/data.ts`. Blog posts come from the MDX content pipeline.
- **Path alias.** `@/*` maps to `./src/*` (configured in both `tsconfig.json` and `vite.config.ts`).
- **Manual chunking.** `vite.config.ts` splits vendor bundles into `react-vendor`, `animation` (framer-motion), and `icons` (lucide-react).

---

## Directory Structure

Only folders that exist are listed.

- `content/` — MDX blog posts, grouped by topic folder: `python/`, `react/`, `web/`.
- `docs/` — Project documentation (`ARCHITECTURE.md`, `CHANGELOG.md`, `CONTRIBUTING.md`, `DECISIONS.md`, `PROJECT_CONTEXT.md`, `ROADMAP.md`, `SECURITY.md`).
- `public/` — Static assets copied verbatim into the build: `404.html` (SPA redirect), `CNAME`, `favicon.ico`, `favicon.svg`, `robots.txt`, `sitemap.xml`.
- `src/` — Application source.
  - `components/` — Reusable UI components (see **Components**).
  - `pages/` — Route-level page components.
  - `hooks/` — Custom React hooks (`src/hooks/index.ts`): `useScrollPosition`, `useMediaQuery`, `useScrollToTop`.
  - `lib/` — Static data and the MDX content loader (`src/lib/data.ts`).
  - `utils/` — Pure helpers: `helpers.ts` (`formatDate`, `formatDateShort`, `paginate`, `getTotalPages`) and `blogImages.ts` (cover-image resolution and generated SVG fallbacks).
  - `styles/` — `globals.css` (Tailwind layers + brutalist component classes).
  - `types/` — TypeScript interfaces (`index.ts`) and MDX module typings (`mdx.d.ts`).
- `.github/workflows/` — CI/CD (`deploy.yml`).

---

## Routing

Routing uses `react-router-dom` v6. `App.tsx` declares all routes in a `<Routes>` block (no nested layout routes / `<Outlet>`). Defined routes:

- `/` → `Home`
- `/blog` → `Blog`
- `/blog/:slug` → `BlogDetails`
- `/about` → `About`
- `/contact` → `Contact`
- `/categories` → `Categories`
- `/newsletter` → `Newsletter`
- `/privacy-policy` → `PrivacyPolicy`
- `/accessibility` → `AccessibilityStatement`
- `*` → `NotFound`

All page components are lazy-loaded and wrapped in `Suspense` with a `LoadingSpinner` fallback, keyed by `location.pathname` for transition animations. Deep links are supported on GitHub Pages through `public/404.html`, which encodes the path into a query string that the inline script in `index.html` decodes on load.

---

## Content System

- Blog posts are authored as **MDX** files under `content/<topic>/<slug>.mdx`. There are 9 posts (5 under `python/`, 2 under `react/`, 2 under `web/`).
- Each post begins with YAML frontmatter (`id`, `title`, `excerpt`, `date`, `category`, `author`, `authorAvatar`, `readingTime`, `image`, `featured`, `tags`, and optional `draft`), parsed via `remark-frontmatter` + `remark-mdx-frontmatter`.
- `src/lib/data.ts` builds `blogPosts` from the compiled modules, filters out any post marked `draft`, and resolves each cover image through `resolveBlogCoverImage` (remote/local resolution with a generated SVG fallback in `src/utils/blogImages.ts`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sahaya-savari/sahaya-savari.github.io](https://github.com/sahaya-savari/sahaya-savari.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
