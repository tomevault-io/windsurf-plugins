---
trigger: always_on
description: Purpose: give an AI coding agent the minimal, actionable knowledge to be productive in this Hugo-based CTF blog repository.
---

# hlc23.github.io — Quick AI agent guide

Purpose: give an AI coding agent the minimal, actionable knowledge to be productive in this Hugo-based CTF blog repository.

- Project type: Hugo static site (Hugo extended) with two themes: `enchanted-lowkey` (blog) and `reveal-hugo` (presentations). Themes live under `themes/` as git submodules — do NOT edit them directly.
- Languages: bilingual site (default `zh-tw`). Content files commonly appear as `index.md` (zh-tw) and `index.en.md` (English).

- Quick dev commands (Windows PowerShell):
  - npm install  # install PostCSS/Tailwind tooling
  - hugo server  # local dev with live reload
  - hugo --minify  # production build -> outputs to `public/`

- Important config & integration points (specific files):
  - `config/_default/hugo.toml` — base config, theme array, goldmark `unsafe = true` for Reveal slides
  - `config/_default/menus.toml`, `menus.en.toml` — navigation per-language
  - `config/tailwind.config.js` and `config/postcss.config.js` — Tailwind & PostCSS pipeline used by Hugo Pipes
  - `layouts/partials/head/styles.html` — concatenates and processes CSS via Hugo Pipes
  - `assets/css/custom.css` (override theme styles) and `assets/css/spoiler.css` (project overrides)
  - `layouts/_markup/render-heading.html` — custom heading renderer that injects anchor links
  - `layouts/shortcodes/spoiler.html`, `layouts/shortcodes/alert.html` — common shortcodes used across posts

- Content & archetypes: use `hugo new` to scaffold. Archetypes live under `archetypes/` (e.g. `archetypes/posts.md`, `archetypes/slides.md`). Examples:
  - Blog post: `hugo new content/posts/my-post/index.md` (edit front matter: `draft:false`, `tableOfContents:true/false`)
  - Reveal slide bundle: `hugo new content/slides/my-presentation/_index.md` and set `outputs = ["Reveal"]`

- Reveal.js specifics:
  - Slides require front matter `outputs = ["Reveal"]` and site Goldmark `unsafe = true` (see `config/_default/hugo.toml`).
  - Use `---` for horizontal slides and `___` for vertical sections.

- Build & deploy notes:
  - Repo has GitHub Actions deploy workflow at `.github/workflows/hugo.yml` (builds with Hugo extended and publishes to Pages).
  - Themes are submodules — update via `git submodule update --remote --merge` when necessary.

- Conventions & patterns to follow (discoverable from repo):
  - Images and auxiliary files live next to each post's `index.md` (content bundles). Use relative paths in markdown.
  - Keep UI/style overrides in `assets/css/custom.css` instead of editing `themes/`.
  - Tailwind purge scans content and theme files; add new dynamic classes to `config/tailwind.config.js` if they are stripped in production.

- Quick checks before editing/building:
  1. Run `npm install` once (PostCSS/Tailwind). 2. Confirm Hugo extended is available locally. 3. For slide changes, ensure `unsafe=true` in goldmark.

If anything above is unclear or you want a shorter or longer agent brief, tell me which sections to expand or trim and I will iterate.
# hlc23.github.io - AI Coding Agent Instructions

## Project Overview
This is a **bilingual Hugo static site** (繁體中文/English) for a CTF blog with integrated Reveal.js presentation support. The site uses two Hugo themes: `enchanted-lowkey` (main blog) and `reveal-hugo` (presentations).

## Architecture

### Dual Theme Setup
- **Primary theme**: `enchanted-lowkey` - customized blog theme with Tailwind CSS
- **Secondary theme**: `reveal-hugo` - for creating presentation slides
- Both themes coexist; content type determines which renders
- Theme array in config: `theme = ['enchanted-lowkey', 'reveal-hugo']`

### Directory Structure
```
config/_default/        # Split Hugo configuration
  hugo.toml            # Base config (baseURL, languages, theme)
  menus.toml           # zh-tw navigation menu
  menus.en.toml        # English navigation menu
  params.toml          # Site parameters (avatar, favicon, comments)
content/
  posts/               # Blog posts (CTF writeups, markdown guides)
    */index.md         # Each post is a directory with resources
  slides/              # Reveal.js presentations (unused currently)
  about/               # About pages per language
layouts/
  shortcodes/          # Custom Hugo shortcodes
  _markup/             # Custom markdown renderers
  partials/head/       # CSS/JS injection points
assets/css/            # Source CSS files (processed by Hugo Pipes)
config/                # PostCSS & Tailwind configs
themes/                # Git submodules (DO NOT EDIT DIRECTLY)
public/                # Generated static site (gitignored)
i18n/                  # Translation strings per language
```

## Critical Developer Workflows

### Local Development
```bash
# Install dependencies (required once)
npm install

# Start Hugo dev server with live reload
hugo server

# Build for production (outputs to public/)
hugo --minify
```

### Creating New Content
```bash
# New blog post (uses archetypes/posts.md template)
hugo new content/posts/my-new-post/index.md

# Edit front matter to publish:
# draft: false
# tableOfContents: true/false
# categories: [CTF, Programming, etc.]
# tags: [Write Up, Tutorial, etc.]

# New Reveal.js presentation (single file)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hlc23) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
