---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

The Austin Mesh website — `www.austinmesh.org`. Built with Astro + MDX, deployed as a fully static site to Cloudflare via Workers Static Assets (Workers Builds, not the legacy Pages product). Page content is authored in markdown/MDX across two content collections (`pages` and `projects`); the layout, head/OG tags, header, nav, and footer are centralized so adding a new page or project is one MDX file.

Hard constraint to honor: **no client-side framework runtime**. The only client JS the build ships is (a) the ~30-line vanilla handler on `/search/` that calls Pagefind's JS API and renders results into matcha-native markup, and (b) the inline `onclick="document.getElementById('event').showModal()"` on the event-dialog trigger. Don't add React/Vue/Svelte hydration; don't add page-wide JS scripts.

## Common commands

- `npm run dev` — Astro dev server on http://localhost:4321
- `npm run build` — production build (includes `pagefind --site dist` postbuild for the search index)
- `npm run preview` — preview the built site locally
- `npx astro sync` — regenerate content-collection types after editing `src/content/config.ts`

## Architecture

- **Pages live in `src/content/pages/`** as `.mdx` files. Frontmatter is validated against the Zod schema in `src/content/config.ts` (`title`, `description` required; `ogImage`, `ogImageAlt`, `canonical`, `eventDialog`, `pagefind`, `publishedAt` optional). The URL is derived from the file path — `src/content/pages/about/meshcore-2-byte.mdx` → `/about/meshcore-2-byte/`. Top-level routes like `/devices/` come from `devices.mdx`.
- **Projects are a second collection** at `src/content/projects/` with its own schema (adds required `thumbnail` + `thumbnailAlt`, optional `author`). They have dedicated routes: `src/pages/projects/index.astro` renders the browse grid, and `src/pages/projects/[...slug].astro` renders each writeup. Per-project pages fall back to the thumbnail as the OG image when `ogImage` isn't set in frontmatter.
- **Routing is a single catch-all** at `src/pages/[...slug].astro` that renders any entry in the pages collection through `BaseLayout`. Trailing-slash directory-style URLs are pinned via `trailingSlash: 'always'` and `build.format: 'directory'` in `astro.config.mjs`.
- **Stand-alone Astro pages** live in `src/pages/`: `search.astro` (ships the ~30-line vanilla Pagefind handler — no `pagefind-ui.*` bundle, excluded from the sitemap) and the `projects/` directory (uses BaseLayout but is not in the pages collection).
- **BaseLayout owns the `<head>`**: title, description, canonical, full OG/Twitter set, favicons, manifest, sitemap link. The default `og:image` is `src/assets/hero/austin-mesh-wildflower-center-large.webp`, processed through `getImage()` to produce a hashed, absolute URL. Pages override via `ogImage` in frontmatter. A `<meta name="robots" content="noindex, nofollow">` is conditionally emitted when `process.env.WORKERS_CI_BRANCH` is set to anything other than `main` (so Workers Builds preview deploys aren't indexed; see Deploy).
- **`pagefind` and `eventDialog` are opt-out per page** (defaults true). `pagefind: false` omits the `data-pagefind-body` attribute on `<main>` so the page isn't indexed (`privacy.mdx` uses this). `eventDialog: false` hides the meet button + modal (also `privacy.mdx`). The search page hard-codes `pagefind: false`.

## Working with content

- New page → one `.mdx` file under `src/content/pages/`. Required frontmatter is `title` + `description`. URL follows the path.
- New project writeup → one `.mdx` file under `src/content/projects/`. Required frontmatter is `title` + `description` + `thumbnail` + `thumbnailAlt` (optional `author`). The file appears automatically in the `/projects/` grid; no nav edit needed.
- Adding a page to the nav requires editing `src/components/Nav.astro` (the menu is hand-listed, not auto-generated).
- Images used in MDX must live under `src/assets/` and be imported as ES modules so Astro can fingerprint, resize, and emit `srcset` via `<Image>` from `astro:assets`. Don't reference `/images/...` from MDX — that path doesn't exist post-migration. Import depth: pages at `src/content/pages/*.mdx` use `../../assets/...`; pages one level deeper (`src/content/pages/about/*.mdx`, `src/content/pages/join/*.mdx`) use `../../../assets/...`.
- The wildflower-center hero pair lives at `src/assets/hero/` and the logo SVGs at `src/assets/logo/`. Everything else under `src/assets/` mirrors the old `/images/` layout.
- Anything that must keep an exact URL (favicons, manifest, `_redirects`) lives in `public/` and is copied byte-for-byte.

## Redirects

`public/_redirects` is read by Cloudflare Workers Static Assets (same syntax as the legacy Pages `_redirects`). It holds 301s for legacy URLs (old `/faq/`, `/solar/`, `/coverage-map/` paths, plus pages that have since moved between collections — see the file for the current list). Add more in the same format. Hash fragments after the destination are resolved client-side after the 301.

## Discord event automation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [austinmesh/www](https://github.com/austinmesh/www) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->
