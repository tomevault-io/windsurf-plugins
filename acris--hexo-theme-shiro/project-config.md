---
trigger: always_on
description: Human docs: `README.md` / `README_CN.md`. Design tokens: `DESIGN.md`. User chat overrides this file; nearest nested `AGENTS.md` wins.
---

# AGENTS.md

Human docs: `README.md` / `README_CN.md`. Design tokens: `DESIGN.md`. User chat overrides this file; nearest nested `AGENTS.md` wins.

## Project overview

Shiro (白) is a clean, minimalist, multilingual Hexo theme: Nunjucks templates, Tailwind CSS v4, optional MathJax, word count (host plugin), Pagefind search, comments, analytics, and minimal client JS for static output.

## Setup commands

| Command         | Purpose                                                            |
| --------------- | ------------------------------------------------------------------ |
| `npm install`   | Install dev dependencies                                           |
| `npm run dev`   | Tailwind watch (unminified `source/css/style.min.css`)             |
| `npm run build` | Release assets: core CSS, optional `*.min.css`, browser `*.min.js` |
| `npm test`      | Node 24 unit tests + real Hexo/Nunjucks render smoke test          |

- Both `dev` and `build` read `source/css/_tailwind.css` → `source/css/style.min.css`.
- After changing `_tailwind.css`, `source/css/_src/*`, Tailwind utilities in templates, or `source/js/_src/*`: run **`npm run build`** (see Testing for committing outputs).
- Do **not** hand-edit `source/css/style.min.css`, `source/css/*.min.css`, or `source/js/*.min.js`; do not delete generated CSS or the package lock without clear reason.
- Prefer `npm run build` over `npm run dev` for one-shot validation.

## Repository map

| Path                       | Role                                                                                                                                                                                                                                                                                                                  |
| -------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `layout/`                  | Nunjucks: `_layout.njk` shell (feature gates + foot scripts; include scope does not leak `{% set %}`); `_macro/`; `_partial/common/` (`head`, `header`, …), components, comments, analytics; pages                                                                                                                    |
| `scripts/`                 | Hexo helpers/filters: thin `helpers.js` registrar; `nunjucks.js` layout-root renderer; pure logic in `scripts/lib/` (`html-analysis`, `toc`, `urls`, `seo`, `fonts`, `seal`, `util`); also `mathjax.js`, `images.js`, `pagefind.js`, `word_count.js`                                                                  |
| `scripts/lib/`             | Pure modules required by `helpers.js` / `mathjax.js` / `images.js` (and unit tests): urls, analysis, toc, seo, code-blocks, image-content, image-meta, … Side-effect free — safe if Hexo also loads nested `scripts/**` files.                                                                                        |
| `source/css/_tailwind.css` | Tailwind entry (`@import` core parts) → `style.min.css`                                                                                                                                                                                                                                                               |
| `source/css/_core/`        | Core theme CSS parts: tokens, base, components, dark, theme-toggle (imported by `_tailwind.css`)                                                                                                                                                                                                                      |
| `source/css/_src/`         | Feature CSS → `source/css/*.min.css` (code, toc, search, comments, lightgallery, giscus). Site-cascade files normally wrap rules in `@layer components` (match `style.min.css`); LightGallery and Gist overrides stay unlayered to outrank their unlayered vendor CSS, and the giscus iframe theme is also unlayered. |
| `source/js/_src/`          | Client sources → `source/js/*.min.js` (Hexo ignores `_src` via underscore prefix). Runtime and LightGallery are single-IIFE sources (`runtime.js` / `lightgallery.js`).                                                                                                                                               |
| `tools/`                   | `build-assets.js` (Tailwind + lightningcss + terser minify)                                                                                                                                                                                                                                                           |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Acris/hexo-theme-shiro](https://github.com/Acris/hexo-theme-shiro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
