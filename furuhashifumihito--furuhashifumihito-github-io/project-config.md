---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Jekyll-based bilingual (Japanese/English) academic research homepage for a University of Tokyo graduate student. Deploys to GitHub Pages (user site) at `https://furuhashifumihito.github.io/` via the GitHub Actions workflow in `.github/workflows/pages.yml`.

## Build Commands

```bash
# Install dependencies
bundle install

# Local development server (http://localhost:4000)
bundle exec jekyll serve

# Build static site to _site/
bundle exec jekyll build
```

## Architecture

### Jekyll Structure
- `_layouts/` - Template hierarchy:
  - `default.html` (base)
  - `home.html` extends `default`
  - `publications.html` (plural, list page) extends `default` — renders `site.data.bibliography`
  - `publication.html` (singular, detail page) extends `default` — rendered for each BibTeX entry at `/projects/<key>/`
- `_includes/nav.html` - Bilingual navigation component
- `_data/i18n.yml` - Bilingual labels keyed by `ja` / `en` (accessed via `site.data.i18n[page.lang]`)
- `_data/diary.yml` - Diary entries (daily memo style) with `date`, `text_ja`, `text_en` fields
- `publications.bib` - **Single source of truth for the publications list page** (BibTeX)
- `_plugins/bibtex_publications.rb` - Jekyll generator that parses `publications.bib` at build time, populates `site.data.bibliography`, and emits one virtual detail page per entry at `/projects/<key>/`
- `_publications/<bibtex_key>/` - Per-paper sidecar folder (one folder per paper) holding `meta.yml` (graphical abstract + figures), optional `body_ja.md` / `body_en.md` (Notes body), and image files. Not a Jekyll collection — read manually by the plugin.

### Hero Banner Slideshow

The top banner on the home layout (`_layouts/home.html`) is an auto-advancing
slideshow driven by files placed in `assets/images/hero/`. To add or reorder
slides, just edit the folder contents — no template or config changes needed.

- **Supported extensions**: `.jpg`, `.jpeg`, `.png`, `.webp`
- **Order**: alphabetical by filename. Use a numeric prefix
  (`01-`, `02-`, ...) to control it.
- **Single image**: rendered statically (the rotation JS is a no-op).
- **Multiple images**: crossfade every 5 s (1 s fade). Respects
  `prefers-reduced-motion`.
- **Performance**: the first slide gets `loading="eager"` +
  `fetchpriority="high"` for LCP; the rest are `loading="lazy"`.

The slideshow is assembled at build time by Liquid iterating over
`site.static_files` filtered to `/assets/images/hero/`. Styling lives in
`style.css` under `.hero__banner` / `.hero__slide`; the rotation script is
inlined near the bottom of `_layouts/home.html`.

### Bilingual System
- Japanese pages: `index.html`, `publications.html`, `diary.html`
- English pages: suffix `-e.html` (e.g., `index-e.html`, `publications-e.html`)
- Templates use `page.lang` variable for conditional content:
  ```liquid
  {% if page.lang == 'en' %}English{% else %}日本語{% endif %}
  ```

### Publications — Data Flow

The publications system is driven by a single BibTeX file plus an
optional per-paper sidecar folder. Everything is processed by one
generator (`_plugins/bibtex_publications.rb`).

**1. The list page (`publications.html` / `publications-e.html`)**

`publications.bib` is the single source of truth. At build time the
generator parses it, normalizes each entry, and populates
`site.data.bibliography`. `_layouts/publications.html` iterates over
that array. Adding or editing an entry on the list page is purely a
matter of editing `publications.bib`.

Each entry is normalized to:
```yaml
key:      "furuhashi2025dgpinn_itsc"    # BibTeX entry key
slug:     "furuhashi2025dgpinn_itsc"    # URL slug
title:    "Paper Title"
authors:  "Author Names"                # "Last, First and ..." → "First Last, ..."
venue:    "..."                         # journal / booktitle / howpublished / ...
year:     2025
type:     "conference"                  # journal | conference | domestic
links:    { paper: "...", pdf: "...", github: "...", ... }
abstract: "Abstract text"
bibtex:   "@inproceedings{...}"         # pretty-printed BibTeX block
lang:     "ja"                          # ja | en (from BibTeX `lang` field)
url:      "/projects/<slug>/"
```

Category resolution (`type` field):
1. If the BibTeX entry has a `bib_category = {journal|conference|domestic}`
   field, that value wins.
2. Otherwise, inferred from the BibTeX entry type:
   - `@article` → `journal`
   - `@inproceedings`, `@conference` → `conference`
   - `@misc`, `@techreport`, `@unpublished`, other → `domestic`

Entries are sorted by year descending at build time; within a year, the order
in the `.bib` file is preserved. The list page renders with `<ol reversed>`.

**2. Per-paper detail pages (auto-generated)**

Every BibTeX entry automatically gets a detail page at `/projects/<key>/`,
rendered by `_layouts/publication.html`. No hand-written page is needed —
the plugin creates a virtual `Jekyll::Page` per entry from the normalized
record above.

**3. Per-paper sidecar folder: `_publications/<bibtex_key>/`**

For papers that need extra content (graphical abstract, figures, notes,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FuruhashiFumihito/furuhashifumihito.github.io](https://github.com/FuruhashiFumihito/furuhashifumihito.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
