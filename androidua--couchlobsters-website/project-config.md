---
trigger: always_on
description: This file tells Claude Code everything it needs to know about this project.
---

# Couch Lobsters Website — Project Context for Claude Code

This file tells Claude Code everything it needs to know about this project.
Place this file in the root of the `couchlobsters-website` repo.

---

## What This Project Is

A static podcast website for **Couch Lobsters** — a film & TV series podcast hosted by
Jess & Dima. Built as plain HTML/CSS/JS (no frameworks, no build step required).

**Live domain:** couchlobsters.com  
**GitHub repo:** https://github.com/androidua/couchlobsters-website  
**Hosting:** Cloudflare Pages (auto-deploys when changes are pushed to `main` branch)  
**Workflow:** Edit files locally → `git push` → Cloudflare auto-deploys within ~2 minutes

---

## File Structure

```
couchlobsters-website/
├── index.html          ← Homepage (hero, concept, latest episodes, platform links, upcoming episode teasers)
├── episodes.html       ← All episodes page (full grid of all episodes)
├── watching.html       ← What We're Watching page (filterable grid: status × person × year)
├── about.html          ← About page (show description + host bios)
├── style.css           ← All styles (dark cinematic theme, gold accent #e8c96d)
├── episodes-data.js    ← Episode data array + UPCOMING_EPISODES teaser config (data only — no functions)
├── watching-data.js    ← Watching picks — auto-generated from Google Sheets (do not edit manually)
├── main.js             ← Nav toggle + episode/teaser card rendering + watching page logic + all helpers
├── logo.jpg            ← Self-hosted podcast logo (640×640 JPEG) — nav/hero/footer images + og:image on all pages
├── favicon.jpg         ← Self-hosted podcast logo (300×300 JPEG) — used as <link rel="icon"> on all pages
├── favicon.ico         ← ICO binary (16×16 + 64×64 PNG) — required for Safari's /favicon.ico domain lookup
├── robots.txt          ← Allow all + sitemap pointer
├── sitemap.xml         ← All 4 pages with <lastmod> (episode sync auto-refreshes / and /episodes.html)
├── _headers            ← Cloudflare Pages security headers (CSP, X-Frame-Options, etc.)
├── .github/workflows/sync-watching.yml  ← 6-hourly GitHub Action: Google Sheets CSV → watching-data.js
├── .github/workflows/sync-episodes.yml  ← Daily GitHub Action: RSS → episodes-data.js + follow-up issue
├── .github/workflows/validate.yml       ← CI guardrail: validates data files + HTML invariants on every push
├── .github/scripts/sync_episodes.py     ← RSS parser (fills Apple links via iTunes API, updates sitemap)
├── .github/scripts/validate.js          ← Validation script shared by CI and both sync workflows
└── CLAUDE.md           ← This file
```

---

## How the Site Works

- **No build tools.** Pure HTML, CSS, JavaScript. No npm, no webpack, nothing to install.
- **Episodes are data-driven.** All episode info lives in `episodes-data.js` as a JS array called `EPISODES`.
  Both `index.html` (shows latest 6 on desktop, 4 on mobile) and `episodes.html` (shows all) pull from this same array.
- **Upcoming-episode teasers** are driven by the `UPCOMING_EPISODES` array in `episodes-data.js` (one card per
  entry; `status: "recorded"` shows a "✳ Recorded · Releasing Soon" badge). Set to `[]` to hide the section.
- **Episodes auto-sync daily.** `.github/workflows/sync-episodes.yml` fetches the RSS feed, prepends new episodes
  to `EPISODES`, fills per-episode Apple links via the iTunes Lookup API, refreshes sitemap `lastmod`, and opens
  a GitHub issue listing the remaining manual steps (Spotify per-episode link, teaser update). Spotify episode
  URLs are NOT available via any unauthenticated API — new episodes get the show URL until pasted manually.
- **Episode artwork** is hotlinked directly from the podcast RSS feed CDN (podcloud.fr).
- **What We're Watching** is a filterable page (`watching.html`) showing picks by Jess & Dima.
  Data lives in `watching-data.js` (auto-generated — do not edit manually).
  Managed via Google Sheets; a GitHub Actions workflow (`.github/workflows/sync-watching.yml`) fetches the sheet
  as CSV every 6 hours and commits `watching-data.js` only when content has changed.
  Uses `var WATCHING` (not `const`) — Safari scopes top-level `const` to the declaring script; `var` attaches
  to `window` and is visible across all script tags. Do not change to `const`.
- **CI validation** (`.github/workflows/validate.yml` → `.github/scripts/validate.js`) runs on every push and
  inside both sync workflows before committing: JS syntax, episode/watching data shape, https-only URLs,
  no bare `rel="noopener"`, no inline `onerror=`, no hardcoded episode counts, sitemap well-formedness.
- **Brand images are self-hosted**: `logo.jpg` (640×640, nav/hero/footer + og:image), `favicon.jpg` (300×300,
  `<link rel="icon">` on all pages) + `favicon.ico` (binary ICO in repo root — Safari auto-requests
  `/favicon.ico` for root-domain URLs; without this file Cloudflare Pages serves `index.html` instead, causing
  the tab icon to fall back to a letter). Never hotlink brand images from the Spotify CDN — those URLs rotate.
- **Fonts** are loaded from Google Fonts: Bebas Neue (display), DM Sans (body), Playfair Display (italic accents).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [androidua/couchlobsters-website](https://github.com/androidua/couchlobsters-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
