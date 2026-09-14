---
trigger: always_on
description: This file provides guidance for AI assistants (Claude Code, opencode, etc.) when working with code
---

# CLAUDE.md

This file provides guidance for AI assistants (Claude Code, opencode, etc.) when working with code
in this repository.

## What this repository is

A 12-module, video-first Arabic ML bootcamp companion repo. Each module pairs a YouTube lecture
playlist with written material — Jupyter notebooks (`CODE/`), theory PDFs, and notes. The published
content is notebooks, Python scripts, datasets, and PDFs, plus a `NN_reading.html` per module, one
`OOP_Notes.html` companion page, and a single `index.html` landing page that links the whole course
together. This site layer was added to mirror the structure of an existing SQL-for-data-analysis
course repo: a static HTML "site" with a shared design system (`assets/style.css`), a fixed sidebar
navigating all 12 modules, and GitHub Pages hosting (see `.nojekyll`).

## Site structure (the HTML layer)

Everything renders as plain static HTML opened directly in a browser — no build step, no server, no
package.json. GitHub Pages serves the repo at the site root via `.nojekyll`.

- `index.html` — the landing page. Fixed left sidebar + module cards (one per module), dark/light
  theme toggle. The sidebar appears on every page; the home entry is `index.html`.
- Each module folder contains one `NN_reading.html` (NN = zero-padded module number, e.g.
  `01_reading.html`) that mirrors the lecture topics as written concepts, and a `README.md` with the
  full video table (YouTube links, durations, PDF links, CODE links).
- The notebooks, PDFs, and CSVs inside each module are untouched by the site layer — but each
  module's notebooks are additionally exported to static HTML under `<module>/notebooks/` (see
  "Notebook export" below) so learners can read them in the browser without launching Jupyter.
- **Markdown policy**: `README.md` files are tables-of-contents only, not worth reading as md — link
  to the rendered GitHub view (`https://github.com/a7madmostafa/Arabic_ML_Bootcamp/blob/main/...`,
  `target="_blank" rel="noopener"`) instead of a raw `.md` link. Standalone notes that ARE worth
  reading (e.g. `OOP_Complete_Notes.md`) get converted to styled HTML pages under the shared design
  system (see "Notes to HTML").

## Reading page format

Each `NN_reading.html` links the shared stylesheet — `assets/style.css` from `index.html`,
`../assets/style.css` from module pages — with no other external CSS/JS beyond Google Fonts. Copy
the sidebar/theme-toggle boilerplate and reuse the stylesheet from an existing page rather than
reinventing either; never inline a new `<style>` block.

- **Design system**: fonts are Bricolage Grotesque (headings), Nunito Sans (body), IBM Plex Mono
  (code/labels) via Google Fonts `<link>` tags. Color tokens: ink `#141414`, blue `#2F63E8` /
  blue-dark `#1E4BC4`, gray `#5B6472` / gray-muted `#9CA3AF`, panel `#F3F4F6`, border `#E5E7EB`,
  paper background `#FAFBFC`. Code blocks use VS-Code-dark colors: background `#1E1E1E`, default
  text `#D4D4D4` (**must** be set explicitly on `.code-body`), keyword `#569CD6`, function
  `#DCDCAA`, string `#CE9178`, comment `#6A9955`. Component classes: `.objectives`, `.callout`
  with `.tip`/`.insight`/`.warn`/`.recap` variants, `.code-block` + `.code-header` + `.code-body` +
  `.annotations`, `.compare` with `.good`/`.bad` columns, `.card-grid` (2- or 3-column), `figure`/
  `figcaption`, `.day-links`, `.day-nav` (prev/next navigation cards linking Home ↔ next module).
- **Diagrams**: hand-drawn inline SVG on a shared white + subtle grid background, referencing
  `<pattern id="erd-grid">` (`#e6edf3` 1px grid, declared once in the shared `<defs>` near the top
  of `<body>`). Palette: `#333`/`#1f6fb2`/`#dfe3e8`, flat and crisp, no drop shadows, square corners.
  Use `font-family="Segoe UI, Arial, sans-serif"` on the root `<svg>`. No raster images for
  diagrams — vector only. (The SQL course's largest schema ERDs used Mermaid; this bootcamp's
  reading pages don't need full-schema ERDs, so prefer hand-drawn SVG everywhere.)
- **Code blocks**: when illustrating Python, syntax token spans use `.kw`/`.fn`/`.str`/`.com`
  classes from the SQL design system (they match VS-Code-dark colors regardless of language).
- **Notebook links**: in Module 02–style "lecture-order" pages, each section's first code header
  (`<span class="fname">NN-name.ipynb</span>`) is wrapped in an anchor to
  `notebooks/<same NN>-<actual name>.html` — match by the leading `NN-` number, since the reading
  page's snake_case name differs from the real filename. Likewise the intro "Before you start"
  callout points at the notebooks dir. In Module 03–style page + table layouts, the day-links and
  the `table.api` CODE column link straight to `<module>/notebooks/<subfolder>/<name>.html`.
- **Keyboard `code.inline`** for single identifiers like `pandas`, `CODE/`, `tweets.csv`.
- **Sidebar**: every reading page includes the full 12-module `<nav class="site-sidebar">` with the
  correct module marked `class="active"`, and `../`-prefixed relative paths back to `index.html` and
  the other module folders.
- **Known bug classes to avoid**:
  - `.code-body` needs an explicit base `color` (the default-text token) — otherwise untokened text

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [a7madmostafa/Arabic_ML_Bootcamp](https://github.com/a7madmostafa/Arabic_ML_Bootcamp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
