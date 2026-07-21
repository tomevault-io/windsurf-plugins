---
trigger: always_on
description: Guide for AI coding agents (Claude Code, Cursor, Codex, Aider, etc.) working
---

# AGENTS.md

Guide for AI coding agents (Claude Code, Cursor, Codex, Aider, etc.) working
on this repository. Human contributors will also find it useful.

## 1. Project at a glance

**Baca-Quran.id** is a free, ad-free, analytics-free Qur'an reader for the
web (https://www.baca-quran.id). SvelteKit single-page app, statically
prerendered, deployed to GitHub Pages.

### Non-negotiable philosophy

- **Local-first.** State lives in `localStorage` / `IndexedDB`. No backend,
  no DB, no server.
- **No tracking, no ads, no analytics.** Don't add any.
- **Optional opt-in Firebase sync** for the existing flow only — don't
  wire new features through it.
- **Offline-friendly.** No new third-party HTTP calls without a strong
  reason. Prefer pure-JS solutions and shipped JSON over APIs.
- **No new dependencies** unless a feature is impossible without them.
  Justify additions in the PR description.

If an idea contradicts the above, push back or scope it down.

## 2. Tech stack

| Area          | Choice                                                                       |
| ------------- | ---------------------------------------------------------------------------- |
| Framework     | SvelteKit 2 + **Svelte 5 runes** (`$state`, `$derived`, `$props`, `$effect`) |
| Language      | TypeScript                                                                   |
| Styling       | Tailwind CSS v4 (no `tailwind.config.js`; uses `@tailwindcss/postcss`)       |
| Adapter       | `@sveltejs/adapter-static` — everything prerendered                          |
| Dates         | `dayjs` (Gregorian); `Intl.DateTimeFormat` (Hijri)                           |
| Optional sync | `firebase` — opt-in only                                                     |
| Tests         | None. Verification is `check` + `lint` + `build:ci` + manual.                |
| Package mgr   | **pnpm 10.33+**                                                              |
| Node          | **24.14+** (see `.nvmrc`)                                                    |

**Svelte 5 reminder:** use runes, not `let` reactivity / `$:` blocks.
Component props go through `$props()`, not `export let`.

## 3. Setup

```bash
# Use the pinned Node version
nvm install   # reads .nvmrc → 24.14.1
nvm use

# Install
pnpm install --frozen-lockfile

# Dev server
pnpm dev          # http://localhost:5173

# Verify (run all three before opening a PR)
pnpm check        # svelte-check + tsc
pnpm lint         # prettier --check + eslint
pnpm build:ci     # svelte-kit build (matches CI)
```

If `pnpm` complains `Unsupported environment` it means Node is < 24.14.
`nvm use` first.

The husky pre-commit hook runs `lint-staged` (prettier + eslint --fix on
the staged files). If it fails, fix the underlying issue and re-stage —
don't bypass with `--no-verify`.

## 4. Repository layout

```
src/
  app.css                  Tailwind entry
  app.html                 HTML shell, theme bootstrap
  data/                    Static, shipped JSON / TS data
    surah-data/            Per-surah Arabic + translations
    surah-info/            Per-surah metadata
    daily-doa.ts           Daily prayers
    asmaul-husna.ts
    ayah-of-the-day.ts     Themed seed pool
    islamic-events.ts      Hijri-date events
  lib/                     Reusable Svelte components, stores, utils
    icons/                 SVG icon components (one file each, consistent props)
    translations/          i18n: id.json + en.json + reactive `t` store
    utils/                 Pure-TS helpers (no Svelte)
    ui/                    Generic UI primitives (Button, etc.)
    views/                 Page-level view composites
    *.svelte               Reusable feature components
  routes/                  SvelteKit file-based routes
    +layout.svelte
    +layout.ts             prerender = true; trailingSlash = 'always'
    +page.svelte           Home
    <feature>/+page.svelte ...one folder per feature page
  store/                   Cross-page Svelte stores (state + persistence)
  service-worker.js
scripts/
  routes.js                Static URL list — drives BOTH prerender entries AND sitemap
  makeSitemap.js
  makeTimestamp.js
  verseCountPerSurah.js
.github/
  workflows/ci.yml         Runs on PR: lint + check + build:ci
  workflows/deploy.yml     Runs on push to master: build + push to gh-pages repo
  ISSUE_TEMPLATE/          bug_report.md, feature_request.md
```

### Path aliases (configured in `svelte.config.js`)

| Alias    | Maps to     | Use for                          |
| -------- | ----------- | -------------------------------- |
| `$lib`   | `src/lib`   | Components, utils                |
| `$data`  | `src/data`  | Static data files                |
| `$store` | `src/store` | Cross-page stores                |
| `$app/*` | SvelteKit   | `$app/stores`, `$app/navigation` |

## 5. Conventions cheat sheet

### Adding a new page

1. **Create the route**

   ```text
   src/routes/<slug>/+page.svelte
   ```

   `+page.ts` is **not required** — `src/routes/+layout.ts` already sets
   `prerender = true` and `trailingSlash = 'always'` globally.

2. **Register the URL** in `scripts/routes.js → staticUrls`. This single
   array drives both:
   - prerender entries in `svelte.config.js`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mazipan/baca-quran.id](https://github.com/mazipan/baca-quran.id) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
