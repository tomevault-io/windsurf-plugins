---
trigger: always_on
description: HanZiFun is a static, offline-first Chinese handwriting practice workbook generator for children. The current implementation is an HTML/Tailwind CSS/custom CSS/JavaScript application with an npm-only build toolchain.
---

# HanZiFun Agent Notes

## Project Context

HanZiFun is a static, offline-first Chinese handwriting practice workbook generator for children. The current implementation is an HTML/Tailwind CSS/custom CSS/JavaScript application with an npm-only build toolchain.

Use this directory as the project root for future work:

```text
/Users/rickytan/Code/HanZiFun
```

## Current Implementation

- Entry point: `index.html`
- Tailwind source: `src/tailwind.css`
- Generated Tailwind utilities: `tailwind.css`
- Domain and print styles: `style.css`
- App logic: `app.js`
- Core bundled stroke data: `data/strokes.js`
- On-demand stroke index: `data/stroke-index.js`
- Generated stroke chunks: `data/characters/`
- Production stroke archives: `data/strokes-pack-NNN.zip`
- Vendored ZIP extraction runtime: `vendor/fflate.min.js`
- Content presets: `data/content-templates.js`
- Data/build scripts: `scripts/`
- Production output: `dist/` (ignored by Git)
- Product requirements: `README.md`
- Design philosophy and visual rules: `DESIGN.md`
- Data attribution: `NOTICE.md`

The app preloads 28 core characters and supports all 9574 single-codepoint upstream characters through multiple ZIP packs. Each JSON chunk contains 50 characters; each ZIP pack contains up to 250 characters. The first 3500 come from the first-level common-character table; the remaining characters are upstream single-codepoint data sorted by Unicode code point. HTTP/PWA loads only the pack containing the needed chunk and selectively extracts entries with fflate; `file://` falls back to generated script chunks.

Templates currently implemented:

- Tracing practice
- Stroke-order breakdown
- Blank practice paper
- Article tracing with one visible input character per cell

Tianzi and Mizi are grid styles, not templates. The selected grid style applies to tracing, stroke-order breakdown, blank paper, and article tracing.

## Technical Direction

- Keep the app static and offline-capable.
- Avoid CDN/runtime network dependencies.
- Use Tailwind v4 utilities for application UI layout. Keep Preflight disabled and retain domain-specific paper, SVG, and print rules in `style.css`.
- Prefer SVG for character strokes, grids, arrows, and print-safe rendering.
- Keep print layout accurate in millimeters.
- Support paper sizes A5, A4, A3, and Letter.
- Support portrait and landscape orientation.
- All settings should update the preview immediately. The app should be WYSIWYG.
- Use one-tap segmented controls for short, stable option sets; reserve select menus for long or dynamic lists and number inputs for exact measurements.
- Use minus/input/plus steppers for discrete numeric settings, preserving direct entry, bounds, decimal steps, and mobile touch targets. Keep continuous visual adjustments as sliders.
- Persist settings and optionally input text in `localStorage` with a `settingsVersion`.
- Treat mobile as a first-class responsive target: scaled paper preview, no page-level horizontal scrolling, touch-friendly controls.
- Support PWA installation with a manifest, service worker, app icons, standalone display, and basic offline caching.
- Direct PDF export uses locally vendored jsPDF and native PDF path/line/text primitives. Never rasterize the whole page through Canvas or add a full-page image. Desktop printing may use `window.print()`, while mobile printing should prefer the PDF share/download fallback because some Android browsers ignore direct print calls.
- Built-in content templates include the 3500 common characters in 100-character groups, 100 Tang poems, selected Song Ci poems, the full San Zi Jing plus sections, and selected Shi Jing poems. Regenerate them with `npm run content:data` from a local `chinese-poetry` extract when needed.
- While an on-demand stroke ZIP is loading or decompressing, expose an explicit loading state and render pending grid glyphs with the browser's regular-script fallback stack at a size close to the final SVG strokes.
- After the initial page has been stable for 15 seconds, warm unused stroke ZIP packs in small batches through the Service Worker cache, falling back to low-priority HTML5 `prefetch` hints when no Service Worker is available. Exclude packs involved in the current page, defer while active stroke loads are running, and skip on save-data, 2G, offline, or `file://` contexts.
- Stroke data comes from `hanzi-writer-data`, derived from Make Me A Hanzi. Preserve attribution and license notes when expanding data.

## Product Decisions

- Target users: both parents and teachers.
- Target character coverage: all single-codepoint characters available in `hanzi-writer-data@2.0.1`.
- Do not bundle the full upstream character data into the main payload. A rough `hanzi-writer-data@2.0.1` estimate showed a 3500-character gzip sample around 3.95MB, above the 200KB direct-bundle threshold.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rickytan/HanZiFun](https://github.com/rickytan/HanZiFun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
