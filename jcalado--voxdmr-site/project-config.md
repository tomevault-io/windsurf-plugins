---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

VoxDMR is a landing and documentation site for VoxDMR — a cross-platform app that streams audio to DMR talkgroups (BrandMeister and other DMR networks) via the Rewind protocol. Available as a Rust desktop app for Linux and Windows (GitHub Releases) and as an Android app on Google Play. Built with **Astro** (static output, SSG) using React **islands** for interactivity, Tailwind CSS v4, and Framer Motion (via `motion/react`).

## Commands

- `npm run dev` — Start the Astro dev server on port 3000
- `npm run build` — Static production build to `dist/`
- `npm run preview` — Serve the built `dist/` locally
- `npm run lint` — `astro check` + `tsc --noEmit` (no ESLint)
- `npm run clean` — Remove `dist/`

## Architecture

Static site built with Astro. Content is server-rendered to HTML at build time; only the genuinely interactive pieces hydrate as React islands. Deployed to GitHub Pages at the custom domain (`public/CNAME` → www.voxdmr.com).

**Routing & i18n.** File-based routing under `src/pages/`. i18n is **route-based**: English at the root (`/`, `/radios`, `/docs/<slug>`) and Portuguese under `/pt/…`, configured via Astro's `i18n` in `astro.config.mjs` (`defaultLocale: 'en'`, `prefixDefaultLocale: false`). Each page resolves its `lang` from the route and passes it as a prop; the language switcher is a link to the counterpart-locale URL (no client-side language toggle).

- `astro.config.mjs` — Astro config: static output, i18n, the markdown pipeline (remark/rehype plugins), `@astrojs/react` + `@astrojs/sitemap`, Tailwind via `@tailwindcss/vite`, and the `@` → project-root alias.
- `src/layouts/Layout.astro` — shared `<html>`/`<head>` (meta, OG, favicon), imports global `src/index.css`.
- `src/pages/` — one `.astro` per route, with a `pt/` mirror per locale. `index` (landing), `privacy`, `radios`, `docs/[...slug]` (+ `docs/index` redirect to installation).
- `src/components/` — static `.astro` building blocks (zero JS): `SiteNav.astro`, `CtaBar.astro`, `Footer.astro`, `Logo.astro`, `LanguageSwitcher.astro`, `ScrollReveal.astro`, inline-SVG `icons/`, the landing sections (`landing/HeroSection|Features|UseCases|CtaSection.astro`, `LandingBody.astro`), and the radios banners (`radios/`). `components/docs/` holds the docs React pieces (`DocsShell`, `DocsContent`, `PlatformSwitcher`, `PlatformContext`). Note `Logo`/`LanguageSwitcher` exist as both `.astro` (used by `.astro` pages) and `.tsx` (imported by the docs React island tree).
- `src/islands/` — the only hydrated React: `RadiosGrid` (filter/sort), `ScreenshotGallery` (+ lightbox), `FaqAccordion`, `HeroShot`, and `platformStore.ts` (module-level store syncing the landing's platform toggle across its two islands).
- `src/i18n/` — `en.json`, `pt.json` (flat key→string), `t.ts` (`t(key, lang)` / `getT(lang)`, usable in `.astro` and islands), `routing.ts` (`localeFromUrl`, `altLocalePath`, `localizePath`).
- `src/content/docs/` + `src/content.config.ts` — docs content collection; Markdown at `src/content/docs/{en,pt}/<slug>.md`, rendered via Astro's native pipeline (`render(entry)` → `<Content />`).
- `src/lib/` — `remark-platform.ts` (platform directives, below), `platform.ts` (client platform detect/store), `docs-config.ts` + `docs-i18n.ts` (sidebar/nav + localized titles), `scroll-reveal.ts`.
- `src/docs/docs.css` — docs styles, incl. the `data-platform-filter` show/hide rule and changelog badge styles.
- `src/index.css` — Tailwind v4 `@theme` design tokens + custom utilities (`.soft-shadow`, `.btn-press`, reduced-motion rules).

## Docs authoring: platform-specific content

Platform-specific docs content is marked with **explicit container directives**, rendered through Astro's native markdown pipeline. `astro.config.mjs` wires up `remark-directive` followed by `remarkPlatform` (`src/lib/remark-platform.ts`); the plugin turns the directives into HTML at build time. The docs root carries `data-platform-filter="<active>"` (set on `DocsShell`), and a single CSS rule in `src/docs/docs.css` hides the non-matching elements:

```css
[data-platform-filter="desktop"] [data-platform="mobile"],
[data-platform-filter="mobile"] [data-platform="desktop"] { display: none; }
```

The toggle (`PlatformSwitcher`) persists the user's choice to `localStorage`; the default on first visit comes from UA sniffing (mobile UA → mobile, else desktop).

Markdown doc pages live in `src/content/docs/` (English in `en/`, Portuguese in `pt/`). The conventions are:

1. **Container directives** — wrap platform-only content in a `:::desktop` or `:::mobile` block. `remarkPlatform` renders each as `<div data-platform="desktop|mobile">`, which the CSS rule above shows/hides.

   ```markdown
   :::desktop

   …desktop-only content…

   :::

   :::mobile

   …mobile/android-only content…

   :::
   ```

   - `:::android` is accepted as an alias for `:::mobile` (both map to `data-platform="mobile"`), but prefer `:::mobile` in new content.
   - Put the opening `:::name` on its own line with a blank line after it, and the closing `:::` on its own line with a blank line before it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jcalado/voxdmr-site](https://github.com/jcalado/voxdmr-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
