---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project overview

Personal portfolio for Yankun, a senior product designer. The site is built around a **desk-and-notebook** metaphor with three layers:

1. **Desk scene** (homepage) — an angled, perspective view of a workspace surface with physical objects: a notebook (→ works), a folded paper (→ resume), and a personal object (→ about). Objects are draggable. The desk *is* the navigation — no nav bar.
2. **Canvas / works** (works) — opening the notebook reveals a 12-column editorial plate where projects sit as hand-composed plates with museum-label captions. Same plate system as the desk's 8-col grid, denser column count. Static asymmetry comes from column-span variation, aspect-ratio variation, and vertical baseline alignment within row tracks. Rotation is available as an interaction and compositional tool when it earns its place (e.g., hover enlarge, peeking artifacts). Vertically scrollable spread.
3. **Project detail** (case study) — clicking a project cluster zooms in and opens a vertical-scroll editorial layout. Long-form, well-paced, freed from the two-page spread constraint.

The portfolio itself is a product demonstration.

**Design evolution:** Field Notebook was selected over Liner Notes. The paginated book format was then replaced with the desk + canvas + vertical scroll model to make the experience more interactive and less like a digital magazine. See `00-brief/decisions-log.md` and `00-brief/prd-homepage-desk.md` / `00-brief/prd-notebook-canvas.md` for full rationale.

The Field Notebook prototype (in `03-prototype-explore/`) established the visual language. That language (colors, typography, editorial plates, grain) is locked. The display format has evolved beyond it.

## Design source of truth

`00-brief/vibe-guideline.md` is the canonical reference for all visual and voice decisions. Consult it before making any design, styling, or copy decisions.

Key constraints from the guideline:
- Two-color palette only: ink blue (`--ink: #16265e`) on warm off-white (`--paper: #f4f1ea`)
- Typography: Fraunces (display + body), JetBrains Mono (metadata), Caveat (marginalia placeholder)
- No CSS frameworks, no Tailwind, no component libraries — plain CSS with design tokens is part of the craft demonstration
- Motion is weighted and paper-like. Respect `prefers-reduced-motion`. No scroll-jacking. The canvas has shallow spatial depth (not parallax in the scroll-hijacking sense)
- 12-column grid (1280px max, 24px gutter, 60px margin) for canvas and project detail pages. Desk uses an 8-col plate grid. The plate chrome (column rulings, plate metadata, register marks, fig captions) is shared across all surfaces — same system, different column counts.

## Repository structure

Design assets and planning live at the root. The deployable site lives in `site/`.

- `00-brief/` — Design briefs, vibe guideline, architecture plan (not deployed)
- `01-content/` — Raw case study writing as markdown drafts (not deployed)
- `02-assets/` — Design assets: illustrations, exports, source files (not deployed)
- `03-prototype-explore/` — Early HTML prototype explorations (not deployed, reference only)
- `04-reference/` — Visual references: Mary Kim zine, Jackie Zhang, type specimen (not deployed)
- `site/` — The Vite + React project (deployable)

## Build and development

The site uses **Vite + React + TypeScript** as a single-page application.

```bash
cd site
npm install
npm run dev          # Local dev server (Vite HMR)
npm run build        # Production build to dist/
npm run preview      # Preview production build
```

TypeScript strict mode for all code. Self-hosted fonts (woff2) with `font-display: swap`.

## Deploy

Live at **https://yankun.design** (also `https://yankun-portfolio.pages.dev`). Hosted on Cloudflare Pages, project `yankun-portfolio`. DNS is at Cloudflare.

**Deploys are manual.** Cloudflare's dashboard "Connect to Git" flow has a known OAuth pairing bug on this account — the GitHub App installs cleanly, but Cloudflare's callback fails to pair, so there is no auto-deploy on `git push`. Every deploy goes through Wrangler from local.

### Workflow

After making changes, from the repo root:

```bash
cd site
npm run build
npx wrangler pages deploy dist --project-name=yankun-portfolio --branch=main --commit-dirty=true
cd ..
git add .
git commit -m "<your message>"
git push
```

Build + Wrangler ships the live site. `git push` updates the GitHub repo. They are independent — running one does not trigger the other. Order is not strict, but deploying before pushing is the cleaner habit: a failed build gets fixed before a commit lands.

### Wrangler auth

First-time use opens a browser OAuth flow to Cloudflare; stays logged in ~30 days. If `wrangler` reports `Failed to fetch auth token`, run the command again — it retries via OAuth automatically.

### Future fix


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yankundesign/portfolio-2026](https://github.com/yankundesign/portfolio-2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
