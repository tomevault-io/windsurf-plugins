---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Product context

1in7.info is an endometriosis and adenomyosis awareness campaign. The long-term plan is to distribute QR code stickers in cities, meaning users will arrive phone-first on cellular data, often on mid-range devices. **Mobile performance is the top priority** - every feature decision should weigh mobile impact first. Ambient effects, heavy animations, and large assets must be gated behind tablet-or-wider breakpoints or `prefers-reduced-motion` checks.

## Build / Serve / Deploy

```bash
# Build site (outputs to dist/)
python3 build.py

# Build for local serving (rewrites internal URLs to /)
python3 build.py --base-url=/

# Serve locally
python3 -m http.server 8000 --directory dist

# Check all links in README.md
python3 check_links.py

# Check links with options
python3 check_links.py --timeout=15 --workers=5 --retry=2
python3 check_links.py --json          # JSON report
python3 check_links.py --skip-ok       # Only show problems

# Resource data pipeline
python3 parse_readme.py                # One-time: README.md -> data/resources.json
python3 generate_readme.py             # Render data/resources.json -> README.md
python3 generate_readme.py --validate  # Check schema integrity, duplicate URLs
python3 enrich_resources.py            # Fetch OG metadata for entries
python3 enrich_resources.py --dry-run  # Show what would be fetched
python3 enrich_resources.py --section=fertility-resources --force
python3 update_link_status.py          # Run check_links.py and update resources.json
python3 update_link_status.py --dry-run
```

Deployed via **GitHub Pages** through `.github/workflows/deploy.yml` (push to `main` triggers build). Also configured for **Cloudflare Pages** via `wrangler.toml` (project name: `one-in-seven`, custom domain: `1in7.info`). `base_url` in `site.json` points to the GitHub Pages URL; override with `--base-url=` for other deploys.

## Architecture

**Standalone static site** - no framework, no npm, no bundler. A single Python build script (stdlib only) converts Markdown content + HTML templates into static pages. All CSS/JS is vanilla. **Self-hosted Figtree** (OFL 1.1) - no Google Fonts call, no IP leak.

The site went through a full UI/UX overhaul (Phases 1–5) documented under `design/`. The current state is "v2": three-journey IA, mobile-first CSS, semantic token layer, off-canvas sidebar + top bar + bottom nav shell.

### Build pipeline (`build.py`)

| Function | Purpose |
|---|---|
| `load_config()` | Read `site.json` |
| `parse_frontmatter(text)` | Split `---` frontmatter from body; supports `title`, `description`, `date`, `lastmod`, `draft`, `tags`, `keywords`, `search`, `toc`. Keys may contain hyphens (e.g. `pub-date`) |
| `md_to_html(text, base_url)` | Markdown→HTML: headings (auto-IDs), bold/italic, links (external→`target="_blank"`, internal rewritten with `base_url`), images, lists, tables, code blocks, blockquotes, hr, raw HTML passthrough |
| `build_css_vars(cfg)` | Emit `<style>` block: legacy color tokens + semantic layer from `site.json:semantic` (light/dark/constant/scale) |
| `discover_languages()` | Read supported languages from `translations.json` (excluding `en`), overlay any `content/translations/{lang}/*.md` files. Returns `{"it": {"quiz", ...}, "de": set(), ...}` |
| `load_translated_pages(cfg, lang, en_pages)` | For each English page, load translation from `content/translations/{lang}/{slug}.md` if it exists, otherwise re-render English markdown with the language `content_base` so internal links get the `/{lang}/` prefix |
| `build_hreflang_tags(slug, lang, available_langs, cfg)` | Generate `<link rel="alternate" hreflang="...">` tags including `x-default` pointing to English |
| `build_sidebar_nav(cfg, active_slug, content_base)` | Grouped `<nav>` with `<div class="nav-group">` sections; uses `content_base` for link hrefs (language-aware) |
| `build_footer_links(cfg, content_base)` | Generates footer links using `content_base` for language-aware hrefs |
| `build_structured_data(cfg, page)` | JSON-LD for SEO |
| `build_search_index(pages, cfg)` | `dist/index.json` - title, permalink, summary, content, tags. Excludes pages with `search: false` frontmatter (`/take-action/`, `/privacy/`) |
| `build_sitemap(pages, cfg)` | `dist/sitemap.xml` |
| `build_toc(html, min_headings=4)` | Per-page TOC from H2/H3 (only emitted if ≥4 headings). Suppressed when `toc: false` in frontmatter |
| `minify_css(text)` / `minify_js(text)` | Whitespace/comment stripping |
| `load_pages(cfg)` | Walk `content/`, parse markdown, propagate frontmatter fields (including `search`, `toc`) |
| `render_page(base_tpl, cfg, inner, page_meta, lang, content_base, hreflang)` | Apply `base.html` with all `{{...}}` markers replaced including `{{PAGE_LANG}}`, `{{CONTENT_BASE}}`, `{{HREFLANG_TAGS}}`, `{{PAGE_JS}}` |
| `build_pages_for_lang(...)` | Shared helper: builds search index, homepage, content pages, and 404 for one language. Eliminates duplication between English and translated builds |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bloo-berries/Endometriosis-Adenomyosis-Resources](https://github.com/bloo-berries/Endometriosis-Adenomyosis-Resources) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
