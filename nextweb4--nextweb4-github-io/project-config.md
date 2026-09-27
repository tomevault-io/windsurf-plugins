---
trigger: always_on
description: - This is a generated public runtime tree. Keep `.nojekyll`, `index.html`, `404.html`, `content/site-content.json`, `content/github-projects.json`, `css/site.css`, `js/site.js`, `images/favicon.png`, `images/hero-campus.jpg`, the three README files, this `AGENTS.md`, and `LICENSE` as the complete public artifact.
---

# AGENTS.md

## 1. Project structure
- This is a generated public runtime tree. Keep `.nojekyll`, `index.html`, `404.html`, `content/site-content.json`, `content/github-projects.json`, `css/site.css`, `js/site.js`, `images/favicon.png`, `images/hero-campus.jpg`, the three README files, this `AGENTS.md`, and `LICENSE` as the complete public artifact.
- `NextWeb4/website-source` is the private source of truth; this repository is not a development checkout.

## 2. Run commands
- Preview the committed runtime with `python -m http.server 8000 --bind 127.0.0.1` from the repository root; do not expose the public tree on a LAN interface during review.

## 3. Test commands
- Current runtime tree has no test command; test and artifact validation must be run in private `NextWeb4/website-source` before deployment.

## 4. Build commands
- Current runtime tree has no build command; `python scripts/build-public.py` belongs to private `NextWeb4/website-source`.

## 5. Code style
- The root `LICENSE` is a rights and provenance notice, not an open-source license. Preserve its no-grant language, third-party carve-out, rights holder, and exact contact address; do not replace it with an SPDX license without explicit authorization.
- Keep the exact centered Shields language row with visible labels `English`, `简体中文`, and `日本語` in all three README files.
- Keep README facts, paths, numbers, links, images, and code fences aligned across languages; do not add private-source paths or credentials.
- Preserve UTF-8, semantic HTML, safe text assignment, fixed responsive dimensions, and nonempty image alt text.

## 6. Module boundaries
- Public markup belongs in `index.html` and `404.html`; presentation belongs in `css/site.css`; browser behavior belongs in `js/site.js`; managed overlay data belongs in `content/site-content.json`; read-only synchronized repository metadata belongs in `content/github-projects.json`. Managed presentation is limited to exact 85-115% font scales in 5% steps for `site`, `hero`, `about`, `work`, `skills`, `journal`, and `contact`, `compact`/`standard`/`wide` language-aware primary-heading widths for all six sections, one complete six-section order, and three section-size presets. Each font value affects only its complete module typography; heading widths use module-owned `em` caps and must avoid unintended extra wraps in both Han and English display text.
- A fresh page and 404 render dark before CSS; an explicit stored light/dark preference remains authoritative. Only the hero H1 uses grapheme-safe typewriter motion. `hero_title` is a validated newline-delimited pool (currently 27 unique entries per language) whose first entries are `向世界交付你的价值` and `Deliver your value to the world.`. Keep the complete active accessible name and a stacked reserve for every candidate, type the first entry once after JavaScript initializes even under reduced motion, and pause hidden runs. Activating the title must visibly delete the current graphemes, choose a random non-immediate-repeat entry in memory, then type it. Without JavaScript, render the complete first title immediately; do not show a separate header replay icon.
- The `images/` files are the only runtime media. Admin, backend, tests, deployment scripts, credentials, and private history remain in `NextWeb4/website-source`.
- Documentation describes the generated artifact; authoring changes happen in the private source repository.
- The public tree must preserve the dark-first theme, bilingual first-title entry typing plus delete/random/retype switching, language-aware heading measures, visibility-aware thought rotator, complete live-first GitHub revalidation with truthful cache/synchronized/emergency fallback, exactly four initially rendered project cards plus one keyboard-accessible disclosure for every remaining eligible repository, three sort modes, metadata-derived Chinese descriptions, and source-preserving Topic localization. Each of the four project-driven capability lines must contain at least three unique repositories when the source has three, render exactly three per visible page, and regenerate its introduction, four-Topic rail, and two-Topic evidence metadata from those visible repositories; one repository may validly evidence multiple matching lines. Previous/next/pause and interaction/visibility pauses remain available. Title and project shuffle/expansion states are memory-only and must not enter browser storage or truncate source data. Preserve the one-line wide Chinese photo caption, safe text assignment, equal project-card layout, and wrapping evidence links; these are runtime behavior owned by `js/site.js` and `css/site.css`.

## 7. Prohibited changes
- Do not edit the runtime as a durable content workflow; the next private-source deployment replaces it.
- Do not add `admin/`, `backend/`, `tests/`, `.github/`, `scripts/`, credentials, deploy keys, backups, or private Git history.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NextWeb4/NextWeb4.github.io](https://github.com/NextWeb4/NextWeb4.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
