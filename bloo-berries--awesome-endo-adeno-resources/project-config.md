---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build / Serve / Deploy

```bash
# Build site (outputs to dist/)
python3 build.py

# Build for local serving (rewrites internal URLs to /)
python3 build.py --base-url=/

# Serve locally
python3 -m http.server 8000 --directory dist
```

Deployed via **GitHub Pages** through `.github/workflows/deploy.yml` (push to `main` triggers build). Also configured for **Cloudflare Pages** via `wrangler.toml` (project name: `one-in-seven`, custom domain: `1in7.info`). `base_url` in `site.json` points to the GitHub Pages URL; override with `--base-url=` for other deploys.

## Architecture

**Standalone static site** — no framework, no npm, no bundler. A single Python build script (stdlib only) converts Markdown content + HTML templates into static pages. All CSS/JS is vanilla. **Self-hosted Figtree** (OFL 1.1) — no Google Fonts call, no IP leak.

The site went through a full UI/UX overhaul (Phases 1–5) documented under `design/`. The current state is "v2": three-journey IA, mobile-first CSS, semantic token layer, off-canvas sidebar + top bar + bottom nav shell.

### Build pipeline (`build.py`)

| Function | Purpose |
|---|---|
| `load_config()` | Read `site.json` |
| `parse_frontmatter(text)` | Split `---` frontmatter from body; supports `title`, `description`, `date`, `lastmod`, `draft`, `tags`, `keywords`, `search`, `toc` |
| `md_to_html(text, base_url)` | Markdown→HTML: headings (auto-IDs), bold/italic, links (external→`target="_blank"`, internal rewritten with `base_url`), images, lists, tables, code blocks, blockquotes, hr, raw HTML passthrough |
| `build_css_vars(cfg)` | Emit `<style>` block: legacy color tokens + semantic layer from `site.json:semantic` (light/dark/constant/scale) |
| `build_sidebar_nav(cfg, active_slug)` | Grouped `<nav>` with `<div class="nav-group">` sections, each with a `<button class="nav-group-toggle">` controlling its `<ul>`; sets `aria-current="page"` on the active link |
| `build_footer_links(cfg)` | Generates footer links (About, FAQ, Take action, Privacy) for the sidebar bottom |
| `build_socials(cfg)` | Social-links block |
| `build_structured_data(cfg, page)` | JSON-LD for SEO |
| `build_search_index(pages, cfg)` | `dist/index.json` — title, permalink, summary, content, tags. Excludes pages with `search: false` frontmatter (`/take-action/`, `/privacy/`) |
| `build_sitemap(pages, cfg)` | `dist/sitemap.xml` |
| `build_toc(html, min_headings=4)` | Per-page TOC from H2/H3 (only emitted if ≥4 headings). Suppressed when `toc: false` in frontmatter |
| `build_breadcrumbs(cfg, title)` | Per-page breadcrumb |
| `minify_css(text)` / `minify_js(text)` | Whitespace/comment stripping |
| `load_pages(cfg)` | Walk `content/`, parse markdown, propagate frontmatter fields (including `search`, `toc`) |
| `render_page(base_tpl, cfg, inner, page_meta)` | Apply `base.html` with all `{{...}}` markers replaced |
| `main()` | Clean `dist/`, concat+minify CSS→`dist/css/bundle.<hash>.css`, JS→`dist/js/app.<hash>.js`, build all pages, copy `static/*` (incl. self-hosted fonts) to `dist/`, write `.nojekyll`, `robots.txt`, `sitemap.xml`, `index.json`, `404.html` |

Bundles are content-hashed for cache busting; filenames injected via `cfg["_css_bundle"]` / `cfg["_js_bundle"]` and exposed through `{{CSS_BUNDLE}}` / `{{JS_BUNDLE}}`.

### JS systems

JS modules in `assets/js/` are listed in `site.json:js_files`, concatenated and minified into `dist/js/app.<hash>.js`. Page-shell scripts (sidebar open/close, theme toggle, back-to-top, bottom-nav current-page marker) live inline in `templates/base.html`.

| System | Location | What it does |
|---|---|---|
| Codeblock copy | `assets/js/codeblock.js` | Copy button for `<pre>` blocks |
| Client-side search | `assets/js/search.js` | Loads `/index.json`. Empty-state suggestions (6 i18n keys). Debounced search (200ms). Synonym expansion. Weighted scoring. ↑/↓/Enter/Esc keyboard nav. `aria-live="polite"` on results panel |
| Client-side i18n | `assets/js/i18n.js` | Loads `/i18n/translations.json`, 26 languages, `data-i18n` attributes, localStorage persistence (`site-language`), `dir="rtl"` for Arabic. Exposes `window.__i18nTranslations` for other modules |
| Image carousel | `assets/js/carousel.js` | Auto-rotates 4s. Disabled if `prefers-reduced-motion: reduce` OR `hover: none` (touch). Reacts to live motion-pref changes. ARIA-roledescription + per-slide labels + `aria-hidden` |
| Table → accordion | `assets/js/accordion.js` | Converts tables to accordion layout on mobile via `data-accordion="table"` |
| Sidebar collapse | `assets/js/sidebar.js` | Per-group expand/collapse toggle; state persists to `localStorage["sidebar-groups"]` |
| Take-action copy | `assets/js/take-action.js` | On `/take-action/` only: adds a Copy button to every `<blockquote>` (template scripts for doctor visits, social blurbs). Clipboard API with text-selection fallback. `aria-live="polite"` on label |
| Symptom tracker | `assets/js/tracker.js` | Client-side symptom tracking on `/tracker/`. localStorage only — never transmitted |
| **Inline in `base.html`:** | | |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bloo-berries/Awesome-Endo-Adeno-Resources](https://github.com/bloo-berries/Awesome-Endo-Adeno-Resources) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
