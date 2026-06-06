---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RetroUI is a retro-themed React component library showcase site. It displays copy-pasteable UI components (buttons, cards, inputs, navbars, etc.) and full page templates with a vintage aesthetic. Deployed on Vercel.

## Commands

- **Dev server:** `npm run dev` (Vite)
- **Build:** `npm run build` (Vite production build)
- **Lint:** `npm run lint` (ESLint flat config)
- **Preview prod build:** `npm run preview`

No test framework is configured.

## Architecture

### Tech Stack
React 18 + Vite + Tailwind CSS 3 + Framer Motion. Uses **TanStack Router** (`@tanstack/react-router`) for client-side routing with lazy-loaded pages. Dark mode via Tailwind `class` strategy.

### Design system
"90s zine / faded newsprint" palette. Tokens live in `src/index.css` as CSS variables (`--bg`, `--surface`, `--surface-2`, `--fg`, `--muted`, `--border`, `--border-strong`, `--accent`, `--accent-fg`, `--ok`, `--sticker`). Both themes ship light and dark. Typography: Space Grotesk (sans), JetBrains Mono (mono), Instrument Serif (italic accents). Helper classes: `display`, `display-italic`, `mono`, `label`, `sticker`, `sticker-flat`, `stamp`, `ornament-hr`, `retro-link`, `marquee-track`, `bg-grid`, `bg-dots`, `bg-halftone`, `placeholder-tile`, `nice-scroll`.

### Key Directories

- **`src/components/`** — Retro UI components organized by category (Buttons/, Cards/, Inputs/, Loaders/, Switch/, Tables/, Navbars/, Footers/, Avtars/, SignIn/, Signup/, Pricing/). Also contains `AppUI/` for the site's own shell (Navbar, Footer, PrivacyPolicy, Terms, ContactUs) and `showcase/` for shared showcase layout components.
- **`src/pages/`** — Top-level pages (Home, About, Component/Pages index, ReleaseNotes) plus `ComponentShowcaseRoute.jsx`, the single generic route that renders any `/components/$slug` by resolving the slug against `COMPONENT_REGISTRY` and lazy-loading its data via `componentLoaders`.
- **`src/pagesComponents/`** — Full page template examples (HomePage, AboutPage, DashboardPage, etc.). `PageShowcase/index.jsx` is the `/pages` index; `PageShowcase/PageShowcaseRoute.jsx` is the single generic `/pages/$slug` route.
- **`src/router.jsx`** — TanStack Router config: `createRootRoute` defines the chrome (Navbar + Outlet + Footer + ScrollRestoration + 404). Top-level paths plus two pathless layout routes (both rendering the shared `ShowcaseLayout`) whose children are the generic `/components/$slug` and `/pages/$slug` param routes. All pages are lazy-loaded.
- **`src/data/components/`** — Component variant data by category (`buttonComponents`, etc.); `index.js` exports `componentLoaders` (slug → lazy data loader). Each variant is `{ name, component, code }` with `code` a `?raw` import of the real source.
- **`src/data/pages/`** — Page-template showcase data by slug (`{ slug, title, description, variants }`); `index.js` exports `pageLoaders` (slug → lazy loader).
- **`src/data/releases.js`** — Canonical release log. Source of truth for the `/release-notes` page, the Navbar version marquee, and `NEW`/`UPD` badges on showcase variant chips. See **Release process** below.
- **`src/lib/`** — Shared utilities: `utils.js` (clipboard, `formatComponentName`), `constants.js` (`SITE_META`, `NAV_ITEMS`, `FOOTER_LINKS`, `getComponentRouteMeta`, `getPageRouteMeta`), `registries.jsx` (`COMPONENT_REGISTRY`, `PAGE_REGISTRY` — single source of truth for sidebar, index pages, and the generic showcase routes), `componentStatus.js` (`getComponentStatus(name)` — drives NEW/UPD badges).
- **`src/hooks/`** — Custom hooks: `useCopyToClipboard`, `useScrollToTop`, `useDocumentMeta` (per-route `<title>` + meta tags; restores on unmount).
- **`src/components/AppUI/ErrorBoundary.jsx`** — Class component wrapping `<Outlet />` in the router root layout. Catches render errors with a retro-styled fallback.

### Patterns

- **Adding a new component variant:** Create the component in the appropriate `src/components/<Category>/` folder, then register it in `src/data/components/<category>.js`: add `{ name, component, code }` where `component` is the imported component and `code` is a `?raw` import of the same file (e.g. `import FooSrc from '../../components/Bar/Foo.jsx?raw'`). The displayed copy-paste code is now the real source — there is no hand-maintained code string to keep in sync. If shipping in a release, add its `name` to `components.new` (or `components.updated`) on the next release entry in `src/data/releases.js`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Omi-Patel/retroUI_Component](https://github.com/Omi-Patel/retroUI_Component) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
