---
trigger: always_on
description: > Keep this file concise, current, and action-oriented. Link to repo docs instead of repeating them.
---

# AGENTS.md — Ingeniería Vial TEPATE

> Keep this file concise, current, and action-oriented. Link to repo docs instead of repeating them.

---

## Project Overview

This is the corporate website for **Ingeniería Vial TEPATE, S.A. de C.V.**. It is a static multi-page Astro site for a Mexican B2B road-safety manufacturer/installer. All user-facing content is in Spanish (Mexico), and the visual system is intentionally industrial/brutalist: black surfaces, neon accents, square corners, and monospace details.

The site is not a SPA. Navigation between pages is a full request. Keep changes aligned with the Astro static architecture and avoid client-side routing.

---

## Technology Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Framework / build tool | Astro | 5.x |
| Bundler | Vite | via Astro |
| CSS framework | Tailwind CSS | 4.1.14 (via `@tailwindcss/vite`) |
| Language | TypeScript | 5.7.3 |
| UI islands | React | 18.x (`ContactForm`, `ProductModal`) |
| Runtime target | ES2022 / ESNext modules | — |
| Package manager | npm | — |
| Node engine | 20.x | — |
| Deployment | Vercel | — |
| Analytics | Vercel Analytics (`@vercel/analytics`) | 2.0.1 |

React is used only for isolated Astro islands. Do not convert the site into a React SPA.

---

## Project Structure

Key files and folders:

- [src/pages/](src/pages) for static routes (`index.astro`, `productos.astro`, `servicios.astro`, etc.)
- [src/layouts/Layout.astro](src/layouts/Layout.astro) for shared head/chrome/scripts
- [src/components/](src/components) for Astro components and React islands
- [src/scripts/](src/scripts) for isolated DOM modules
- [src/styles/global.css](src/styles/global.css) as the stylesheet entry point; it imports modular styles from [src/styles/](src/styles)
- [public/](public) for static assets served from `/`
- [vercel.json](vercel.json) for edge headers, CSP, and cache rules
- [AUDIT_REPORT.md](AUDIT_REPORT.md) for remediation history and validation notes
- [FIX_ISSUES_PROMPT.md](FIX_ISSUES_PROMPT.md) for the original fix scope and verification checklist

---

## Build and Development Commands

```bash
# Install dependencies
npm install

# Start development server (port 3000, host 0.0.0.0)
npm run dev

# Production build (outputs to dist/)
npm run build

# Preview production build locally
npm run preview

# Astro + TypeScript check
npm run check

# TypeScript-only check
npm run lint

# Clean build output
npm run clean
```

Important:
- There is no automated test suite in this repo.
- The main code-quality gate is `npm run check` (`astro check && tsc --noEmit`).
- `npm run clean` uses `rm -rf`; on Windows, prefer deleting `dist/` manually if needed.

---

## Build Configuration

### Astro (`astro.config.mjs`)

- **Static output:** `output: 'static'` with file-style HTML output.
- **Routes:** Pages are generated from `src/pages/*.astro`.
- **Sitemap:** `@astrojs/sitemap` generates sitemap output during build.
- **Path alias:** `@/` maps to the project root.
- **Tailwind CSS:** Integrated through Vite with `@tailwindcss/vite`.

### TypeScript (`tsconfig.json`)

- `target`: ES2022
- `module`: ESNext
- `moduleResolution`: bundler
- `isolatedModules`: true
- `noEmit`: true
- `allowImportingTsExtensions`: true
- `paths`: `@/*` → `./*`

---

### Code Organization

### Entry Points

- `src/layouts/Layout.astro` owns the shared document shell, metadata, header/footer slots, Vercel Analytics and rich-motion bootstrap.
- `src/pages/*.astro` owns route content.
- React islands are reserved for stateful widgets such as the quote form and product modal.

### Script Modules / Islands

| Module | Responsibility |
|--------|---------------|
| `analytics.ts` | One-liner wrapper around `@vercel/analytics` `inject()`. |
| `animations.ts` | Optional rich-motion layer. It is skipped for reduced motion, coarse pointers and `/contacto`. |
| `ContactForm.tsx` | B2B quote form. Reads URL intent, validates fields, posts to `PUBLIC_FORMSPREE_ENDPOINT`, and offers WhatsApp/email fallback when not configured. |
| `ProductModal.tsx` | Product detail modal with focus return, Escape close, basic focus loop and quote/WhatsApp CTAs. |
| Inline Astro scripts | Small page-local behavior such as mobile menu, accordions, filters, lightbox and sticky CTA. |

### Styling

The stylesheet is modular now. [src/styles/global.css](src/styles/global.css) imports the layered files under [src/styles/](src/styles); do not reintroduce large inline CSS blocks unless there is a strong reason.

| File | Responsibility |
|------|---------------|
| [src/styles/base.css](src/styles/base.css) | Tokens, reset, focus states, motion reduction, print styles |
| [src/styles/layout.css](src/styles/layout.css) | Shared chrome: topbar, nav, mobile menu, footer, WhatsApp float, back-to-top |
| [src/styles/components.css](src/styles/components.css) | Buttons, cards, forms, accordion, pagination, modal, filters |
| [src/styles/home.css](src/styles/home.css) | Homepage sections |
| [src/styles/pages.css](src/styles/pages.css) | Secondary page layouts and content patterns |

**Design system tokens (excerpt):**
```css
--neon: #E1FF00;        /* Primary accent */
--black: #000000;       /* Page background */

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EJMM17/tepate-web](https://github.com/EJMM17/tepate-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
