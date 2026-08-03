---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

An open-source interactive textbook for physical computing (Arduino, ESP32,
sensors, signal processing, etc.), published at
<https://makeabilitylab.github.io/physcomp/>. It is a **Jekyll static site**
using the [Just the Docs](https://just-the-docs.github.io/just-the-docs/) theme
(pinned to `v0.12.0` via `remote_theme`). Content is almost entirely **Markdown
lesson pages**; there is no application code to build or test in the
conventional sense.

## Commands

All commands must be run from the repo root (where the `Gemfile` lives) —
`bundle` fails with "Could not locate Gemfile" anywhere else.

```bash
bundle install                 # install Ruby gem dependencies (one time)
bundle exec jekyll serve       # build + serve with live reload at localhost:4000/physcomp
bundle exec jekyll serve --no-incremental   # full rebuild if stale theme/content appears
bundle exec jekyll build       # build only, output to _site/
```

Ruby `3.3.11` (see `.ruby-version`). Builds are slow (20s+); `incremental: true`
and `cache: true` are set in `_config.yml` to mitigate this.

### Serving video locally

Jekyll's WEBrick server does not support HTTP range requests, so `<video>`
elements may fail to stream. Build first, then serve `_site/` with a
range-capable server:

```bash
bundle exec jekyll build
python3 -m http.server 4000 --directory _site   # or: npx serve _site
```

### Maintenance scripts (`scripts/`)

Python utilities that bulk-edit or generate content across the site. **All default
to a dry run; pass the apply flag and inspect `git diff` afterward.** Each script
is documented in [`scripts/README.md`](scripts/README.md).

**IMPORTANT: keep `scripts/README.md` up to date.** Whenever you add, remove,
rename, or meaningfully change a script in `scripts/`, update `scripts/README.md`
(its table and details section) in the same commit.

- `check_seo_frontmatter.py` — CI gate (Content lint workflow, runs on PRs): fails if any published page lacks `description:`. Drafts (`nav_exclude`/`search_exclude`) and contributor docs are exempt.
- `generate_og_posters.py [--run] [--force]` — generate static OG/social-card posters from a page's hero `<video>` (MP4) via ffmpeg and set `image:` front matter.
- `update_lesson_nav.py [--run]` — migrate old `.btn` lesson nav to card-style `<nav class="lesson-nav">` (converts `.md` links to `.html`).
- `fix_embedded_media.py [--run]` — normalize `<video>` styles and wrap bare YouTube iframes responsively.
- `fix_arduino_urls.py [--apply] [--verify]` — migrate old `arduino.cc` URLs to `docs.arduino.cc`. Marked untested/brittle in its own header — use with care.

## Content architecture

Top-level directories are the textbook **modules**, each with an `index.md`
landing page and lesson `.md` files: `electronics/`, `arduino/`, `sensors/`,
`advancedio/`, `communication/`, `esp32/`, `signals/`, `cpx/`,
`microcontrollers/`, `resources/`, `tools/`. Module ordering on the site is
controlled by `nav_order` in front matter, not directory names.

Each module folder typically has its own `assets/` subfolder for that module's
images/videos. Shared/site-wide media lives in the top-level `assets/`
(`images/`, `videos/`, `css/`, `datasheets/`, `calculations/`).

### Page front matter conventions

Lesson pages use YAML front matter to drive Just the Docs navigation and
features. Common keys:

- `layout: default`, `title:` — title often rendered via Liquid, e.g. `# {{ page.title | replace_first:'L','Lesson '}}`.
- Nav hierarchy: `nav_order`, `parent`, `grand_parent`, `has_children: true`, `nav_exclude: true`.
- `usemathjax: true` — opt in to MathJax (LaTeX via `$$...$$`); wired up in `_includes/head_custom.html`.
- `usetocbot: true` — opt in to the floating tocbot table of contents.
- `comments: true` — enable comments on the page.

The in-page Table of Contents is the recurring boilerplate block:

```markdown
## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}
---
```

### Theme customization

- `_includes/head_custom.html` — favicon, MathJax config, tocbot CSS, custom CSS link.
- `_includes/footer_custom.html`, `nav_footer_custom.html` — footer overrides.
- `assets/css/custom.css` — all custom styling, organized into numbered sections (referenced by scripts, e.g. "custom.css Section 8" for video styles). Apply a class to a Markdown element with `{: .class-name }` on the line below it.
- Callouts are configured in `_config.yml` (`note`, `tip`, `warning`, `important`, etc.) and used as `{: .note }` above a paragraph.

## Companion code repositories (keep in lockstep)

The code featured in lessons lives in **separate repos**, not here. Lessons link
out to them, and those repos' source files link *back* to specific lesson URLs —
so the two sides must stay in sync.

- Arduino sketches — <https://github.com/makeabilitylab/arduino> (`d:\git\arduino`)
- MakeabilityLab Arduino library — <https://github.com/makeabilitylab/makelab-arduino-lib> (`d:\git\makelab-arduino-lib`)
- Signals (Python / notebooks) — <https://github.com/makeabilitylab/signals> (`d:\git\signals`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [makeabilitylab/physcomp](https://github.com/makeabilitylab/physcomp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
