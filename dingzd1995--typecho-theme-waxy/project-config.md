---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Waxy** is a minimal, responsive Typecho blog theme. Typecho is a lightweight PHP blogging platform. This is a pure PHP theme — no build step, no package manager, no transpilation.

## Development Setup

No build process required. To use the theme:
1. Copy theme files to `/usr/themes/Waxy/` in a Typecho installation
2. Activate via Typecho admin panel: 控制台 → 外观 → Waxy → 启用

There are no tests, no linting tools, and no CI/CD configuration in this repo.

## Architecture

### Template System
Typecho routes requests to PHP templates. All templates follow this pattern:
```php
get_header();        // includes header.php
// ... page-specific content ...
get_sidebar();       // includes sidebar.php
get_footer();        // includes footer.php
```

Template files: `index.php` (post list), `post.php` (single post), `page.php` (static page), `archive.php`, `404.php`, plus special page templates prefixed with `page_` (`page_friends.php`, `page_timeline.php`, `page_articles.php`, `page_articles_month.php`, `page_sitemap.php`).

The post list markup lives in `post_list.php`, included by `index.php`. It renders two modes based on `$this->options->articles_list`: full-article mode (`== 1`) and excerpt/card mode (`== 0`).

Full-article mode shows "阅读全文" from whichever of two triggers fires first. `getIndexContent()` returns `['content' => ..., 'has_more' => bool]`: if the post has a `<!--more-->` marker, content is truncated there (like the old excerpt-index behavior) and `has_more` is true, so `post_list.php` renders the sibling `.readall` block visible immediately (no `hidden` attribute) — the author's marker always wins regardless of rendered height. If there's no marker, the full content is shown and `.readall` starts `hidden`; `.post-content.post__content[data-clamp-total="800"]` is then watched by `initContentClamp()` in `js/waxy-main.js`, which measures the whole `.post` article's rendered `scrollHeight` (via `ResizeObserver`, since lazy-loaded images grow it after initial paint) — if the *whole item* (head + content + footer) would exceed 800px, it adds `.post__content--clamped` (CSS `max-height: 460px`) to just the content box and reveals `.readall`. `.post__content--clamped` sets `overflow-x: visible; overflow-y: clip` rather than plain `overflow: hidden`, so wide content (tables, code blocks) isn't clipped sideways along with the bottom — CSS's "visible pairs with non-visible → becomes auto" rule means a plain `overflow-y: hidden` would silently turn `overflow-x` into a second clipping axis too. Since full-mode list content is no longer guaranteed to be short, `header.php`'s code-highlight gate loads Prism whenever `articles_list == 1`, not just on single/page views (it can't cheaply pre-scan every row in the loop for code fences); `shortcode.php`'s `[poststats]` guard was similarly relaxed to render in full-list mode, not just `is('single') || is('page')`.

Excerpt/card mode is a flex layout: a fixed-ratio cover image (`.excerpt__img`, 5:3 via `aspect-ratio`) beside a body column (`.excerpt__body`) with title → excerpt text → footer. The footer (`.excerpt__info`) holds author/date (`.excerpt__item`) plus one or more badge groups (`.excerpt__badges`): a single leading icon followed by each category/tag rendered as its own `.excerpt__badge` link. Both groups are capped (`$waxy_badge_groups` in `post_list.php`: 2 categories, 3 tags) — items beyond the cap collapse into a `+N` badge (`.excerpt__badge--more`) whose CSS-only hover/focus tooltip (`.excerpt__badge-tip`) lists the rest as clickable `.excerpt__badge-tip__item` links. Badge fill color is `var(--waxy-badge-bg)`, a variable that differs from `--waxy-bg-soft` specifically so it stays visible against the card's `--waxy-surface` background in light mode; the sidebar tag cloud (`.tag-cloud a`) shares this same badge styling for visual consistency.

### Core Files
- **`functions.php`** — All theme functions and the `themeConfig()` hook that registers 30+ admin settings. This is the main logic file.
- **`shortcode.php`** — Registers shortcode handlers; delegates to `lib/shortcode.php` for the engine.
- **`lib/icons.php`** — Inline SVG icon library; use `waxy_icon('name')` or `waxy_icon('name', 'extra-class')` everywhere an icon is needed. Never embed raw SVG in templates.
- **`css/waxy-main.css`** — All custom styles (~3600+ lines); Bootstrap 3 is the base framework loaded via CDN.
- **`js/waxy-main.js`** — Custom JS: lazy loading (IntersectionObserver), lightbox (no deps), back-to-top, dropdown menu, dark mode toggle, mobile drawer.

### Content Processing Pipeline
In `functions.php`, two content functions exist — use the right one:
- `getContent($content)` — for single post/page views; runs shortcodes → `getPicHtml()` → `waxy_process_toc()` (populates `$GLOBALS['waxy_toc_items']` for the sidebar TOC widget)
- `getIndexContent($content)` — for the post list's full-article mode (`articles_list == 1`); runs shortcodes → `getPicHtml()`. It ignores `<!--more-->` and always returns the complete content — truncation is no longer done server-side


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dingzd1995/typecho-theme-waxy](https://github.com/dingzd1995/typecho-theme-waxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
