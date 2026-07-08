---
trigger: always_on
description: A static site showcasing Sandy Brook's internal product experiments and study guides. Sibling to the main Sandy Brook DevWorks site at `sandybrookdevworks.com` (source: `~/Repos/sandy-brook-web`).
---

# Sandy Brook Projects Lab

A static site showcasing Sandy Brook's internal product experiments and study guides. Sibling to the main Sandy Brook DevWorks site at `sandybrookdevworks.com` (source: `~/Repos/sandy-brook-web`).

## Codex Notes

- This `AGENTS.md` file is the repo-local instruction source for Codex.
- Preview the static site from the repo root with `python3 -m http.server 8080`.
- There is no build step, package install, or framework tooling to run for ordinary content changes.
- Keep page-specific styles and scripts external unless this file or `STYLE_GUIDE.md` explicitly says an inline snippet must remain inline.

## Stack

- Plain HTML, vanilla JS, Tailwind CSS v4 via CDN (`@tailwindcss/browser@4`)
- Google Fonts: Inter (body, weights 300–700), JetBrains Mono (code, weights 400, 600)
- No build step, no frameworks, no npm

## File Structure

| File / Directory | Purpose |
|------------------|---------|
| `index.html` | Main Projects Lab page (Relay, KnowItOwl, Aquorbis) |
| `styles/brand.css` | **Shared** — CSS custom properties, brand colors, grid bg, dark mode overrides |
| `styles/theme.js` | **Shared** — `toggleTheme()`, `toggleMenu()`, `closeMenu()` functions |
| `images/` | Root-level brand assets: `blue_logo.jpg`, `dark_logo.jpg`, `relay_logo.png`, `relay_logo.webp`, `cogniwatch_logo.webp` |
| `guides/index.html` | Guides hub page |
| `guides/content.css` | **Shared** — Content styling for all guide pages (typography, callouts, code blocks, tables, etc.) |
| `guides/ai-driven-development/` | AI driven development workflow guide (Markdown source, HTML page, infographic assets) |
| `guides/rag/` | Introduction to RAG guide based on the RAGPipeline sample repo |
| `guides/linq/` | C# Data Structures & LINQ course (index + 5 lessons) |
| `guides/ximena/` | Standalone research guide page; styles in `guides/ximena/styles/` |
| `apps/relay/` | Relay engineering case study page using shared brand styles |
| `apps/aquorbis/` | Aquorbis landing site — see "Aquorbis Structure" below |
| `apps/knowitowl/` | KnowItOwl! landing site — see "KnowItOwl Structure" below |
| `STYLE_GUIDE.md` | **Human-readable design system reference** with templates and how-to guides |
| `AGENTS.md` | Codex project instructions and repo conventions |
| `.claude/launch.json` | Legacy Claude preview server config (python3 http.server on port 8080) |

## Shared Assets

Three files eliminate duplication across all pages:

1. **`styles/brand.css`** — CSS variables (`--brand-teal`, `--font-sans`), body font stack, grid background SVGs, `.dark` utility overrides, mobile bottom sheet menu styles. Linked via `<link rel="stylesheet">`.

2. **`styles/theme.js`** — `toggleTheme()` for dark mode, `toggleMenu()`/`closeMenu()` for the mobile bottom sheet menu. Loaded via `<script src="...">` (no defer). Dark mode *init* is a 1-line inline `<script>` in each page's `<head>` to prevent FOUC.

3. **`guides/content.css`** — Content styling for guide pages. Targets elements inside `<main>` via descendant selectors (`main h1`, `main p`, etc.). Includes callout system (`.callout.tip/warn/danger`), code block styling, table styling, challenge boxes, page nav, syntax highlighting spans, Big-O badges.

Each HTML page also has a minimal inline `<style type="text/tailwindcss">` block (7 lines) for the Tailwind `@theme` and `@custom-variant` directives that the CDN requires inline, plus a 1-line inline `<script>` that applies the `dark` class before paint to prevent FOUC. These two snippets must stay inline; everything else should live in external files under `styles/` or `scripts/`.

## Per-App Asset Layout

Each app under `apps/` keeps its own `styles/` and `scripts/` folders so the apps stay isolated from the brand-wide design system.

### Relay Structure

```
apps/relay/
  index.html      # Case study for the released Relay product; uses root shared brand.css/theme.js
```

Relay's live product site is `https://relayconnect.net/`. The lab page is the Sandy Brook engineering/case-study companion and links to the live signup/demo flow.

### Aquorbis Structure

```
apps/aquorbis/
  index.html
  privacy.html
  support.html
  styles/
    shared.css   # nav, footer, bubbles, body/font setup, app-store button — used by all 3 pages
    index.css    # landing-only: hero, features, zones, fish gallery, growth, aquarium, CTA
    legal.css    # privacy/support shared: page-header, content cards, FAQ, contact-btn
  scripts/
    bubbles.js   # background bubble generator. Reads `data-count` from `.bubbles` (defaults to 20).
  images/        # in-game art, app icon, App Store badge, etc.
```

### KnowItOwl Structure

```
apps/knowitowl/
  index.html
  privacy.html
  support.html
  styles/
    main.css                   # .glass, fade/float animations, FAQ details, print rules
  scripts/
    tailwind-config.js         # Tailwind CDN sage/cream palette + system-font stack. Loaded right after the CDN script.
    theme-toggle.js            # wires up #theme-toggle button + persists to localStorage
    scroll-fade-in.js          # IntersectionObserver fade-up for index.html sections

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sandy-Brook-DevWorks-LLC/sandy-brook-labs](https://github.com/Sandy-Brook-DevWorks-LLC/sandy-brook-labs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
