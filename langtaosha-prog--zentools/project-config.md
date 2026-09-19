---
trigger: always_on
description: Pure static HTML5/CSS3/Vanilla JS site. ~410 HTML pages, 279 tools across 13 categories. Deployed via GitHub Pages from main branch root. No package.json, no build tools, no frameworks.
---

# ZenTools AGENTS.md

## Project Overview

Pure static HTML5/CSS3/Vanilla JS site. ~410 HTML pages, 279 tools across 13 categories. Deployed via GitHub Pages from main branch root. No package.json, no build tools, no frameworks.

## Quick Commands

```bash
# Start local dev server (HTTP server only - no build step)
python3 -m http.server 8000

# Generate new tool page (creates HTML + updates tools-data.json + regenerates sitemap)
python3 _add_tool.py --slug pdf-ocr --category "PDF工具" \
  --name-zh "PDF OCR" --name-en "PDF OCR" --name-ja "PDF OCR" --name-vi "PDF OCR" \
  --desc-zh "描述" --desc-en "Description" --desc-ja "説明" --desc-vi "Mô tả" \
  --keywords "keyword1 keyword2"

# Generate new tutorial (result in /tutorials/)
python3 _add_tutorial.py --slug my-tutorial --category "PDF工具" \
  --title-zh "教程标题" --desc-zh "描述" --tool-url "/pdf/some-tool.html"

# Generate new guide/review (result in /guides/)
python3 _add_guide.py --slug my-review --type review \
  --title-zh "评测标题" --desc-zh "描述" --word-count 2500 --read-minutes 20

# Batch-unify tool pages to current --zen-* design system
# Run after _add_tool.py or any new tool HTML is created
python3 _batch_unify_ui.py              # migrate all tool/category pages to current UI

# Batch-unify tutorial pages to current --zen-* design system
# Run after _add_tutorial.py or any new tutorial HTML is created
python3 _batch_unify_tutorials.py       # migrate all tutorials/ pages to current UI

# Post-edit verification pipeline (run in this order after any data change)
python3 _check_json.py                  # validate JSON integrity
python3 _sync_tools_data_js.py          # JSON -> JS data sync
python3 _gen_sitemap.py                 # regenerate sitemap.xml
python3 _minify_assets.py               # re-minify JS/CSS

# Validate JSON files only
python3 _check_json.py
```

## Architecture

- **Data layer**: `data/tools-data.json` (~666KB) drives all tool rendering. Single source of truth. Categories map to directories.
- **i18n system**: `assets/js/common-i18n.js` (public, `window.ZT_COMMON`) + inline `window.ZT_PAGE` per page. Engine: `ZT.applyLanguage()` in `assets/js/tool-ui.js`. Merge priority: `ZT_PAGE` overrides `ZT_COMMON`. Supports zh/en/ja/vi.
- **PWA**: `sw.js` (cache-first with 5 cache tiers: core, assets, data, html/LRU-200, pages), `manifest.json`.
- **Anti-crash**: `assets/js/anti-crash.js` must load FIRST in `<head>`. Catches global errors, JSON corruption, switches to fallback mode after 5 errors/5s.
- **SEO per tool page**: FAQPage + WebApplication + HowTo schema.org JSON-LD, Og tags, Twitter Card, canonical URL.

## Script load order (critical)

Every page `<head>` must follow this exact order:
1. `anti-crash.min.js` (first, before anything else)
2. Meta/SEO tags, canonical, manifest link
3. `tool-ui.min.css`
4. Page-specific `<style>` block — MUST define `:root` and `.dark` CSS variables (see Design System below). All color/background/border references must use `var(--zen-*)` variables only.
5. AdSense script (`ca-pub-1955887568822472`)
6. FAQPage + WebApplication + (optional) HowTo schema JSON-LD
7. Inline `window.ZT_PAGE` with zh/en/ja/vi keys

At end of `<body>`:
8. `common-i18n.min.js`
9. `tool-ui.min.js`
10. Tool-specific logic

## Key Directories

| Path | Content |
|------|---------|
| `pdf/`, `image/`, `text/`, `dev/`, `audio/`, `video/`, `ai/`, `seo/`, `life/`, `finance/`, `qr/`, `json/`, `tools/` | Tool HTML pages (each dir has its own `index.html` category page) |
| `tutorials/` | Tutorial pages (360). No articles/, blog/, posts/ allowed. |
| `guides/` | Deep guides and reviews (28 pages) |
| `compare/` | Tool comparison landing page |
| `assets/js/` | `main.js` (homepage), `tool-ui.js` (global engine), `common-i18n.js` (shared translations), `anti-crash.js` (error resilience) |
| `assets/css/` | `tool-ui.css` (global), `style.css` (auxiliary) |
| `data/` | `tools-data.json` (source of truth), `categories.json` |
| `pdf_tools/` | Standalone Python PDF scripts (not part of web app) |

## Design System (CSS variables)

**Every page** must define these variables in an inline `<style>` block. The system supports light/dark mode via `.dark` class on `<html>`:

> 💡 Single source of truth: brand color **values** & palette are authoritative in `ZenTools_ZT-DCA_开发守则_v1.0.txt` § 十三「品牌开发守则 V1.0」. This section is the CSS-variable implementation set. When changing primary/radius, **sync both files** to avoid drift.

```css
:root {
  --zen-primary: #0066FF;
  --zen-secondary: #00C2B8;
  --zen-gradient: linear-gradient(135deg, #0066FF, #00C2B8);
  --zen-success: #22C55E;
  --zen-warning: #F97316;
  --zen-danger: #EF4444;
  --zen-text-main: #111827;
  --zen-text-sub: #4B5563;
  --zen-text-placeholder: #9CA3AF;
  --zen-bg-base: #F9FAFB;
  --zen-bg-card: #FFFFFF;
  --zen-border: #E5E7EB;
  --zen-radius-base: 12px;
  --zen-radius-card: 16px;
  --zen-radius-btn: 8px;
  --zen-radius-tag: 6px;
  --zen-card-padding: 20px;
}
.dark {
  --zen-text-main: #F9FAFB;
  --zen-text-sub: #D1D5DB;
  --zen-text-placeholder: #64748B;
  --zen-bg-base: #0F172A;
  --zen-bg-card: #1E293B;
  --zen-border: #334155;
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LANGTAOSHA-prog/ZenTools](https://github.com/LANGTAOSHA-prog/ZenTools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-19 -->
