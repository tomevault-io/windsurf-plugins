---
trigger: always_on
description: TV-first anime streaming hub. Production: https://zenkaitv.com (Vercel auto-deploys on push to `main`).
---

# ZenkaiTV (AnimeTV) — Project Guide

TV-first anime streaming hub. Production: https://zenkaitv.com (Vercel auto-deploys on push to `main`).
Repo: `JSolanoDev/AnimeTV`.

## Scope of work

This is an adults-only media platform. Work only on the software engineering side: UI
architecture, authentication, database, search, media delivery, performance, responsive design,
testing, and deployment.

- Do not inspect or generate sexually explicit media or descriptions.
- Treat media assets as opaque files — referenced by URL/ID, never by their content.
- Use neutral placeholder data during development, in examples, tests, and fixtures.
- "Adults-only" is a hard requirement on the content itself, not just on the audience: any
  source wired into the platform must be one that carries adult-only material. A provider known
  for sexualized depictions of minors (real or drawn) is out of scope regardless of how the
  integration is built.

## Stack — read this before proposing changes

This is a **vanilla-JavaScript app**, deliberately. There is no build step for the app code.

- `client.js` — the whole SPA (~14k lines). Classic script, **global scope**, no modules.
- `js/*.js` — helpers loaded as separate classic `<script defer>` tags (`constants`, `utils`,
  `normalize`, `season-normalization`, `image-resolver`, `adult-mode`, `router`, …).
  They share one global namespace with `client.js`; load order in `index.html` matters.
- `animetv-server.js` — Node HTTP server + all `/api/*` routes. On Vercel it runs as a
  serverless function via `api/[...path].js`.
- `player/` — the video player iframe (ArtPlayer + hls.js from jsdelivr).
- `styles.css` — single stylesheet.
- `scripts/build-static.mjs` — copies `sourceDir = "."` → `dist/` + `public/`, then minifies.

**Do NOT migrate to ESM / Vite / Next.js / TypeScript / Tailwind / shadcn without an explicit,
scoped decision.** Blockers: everything shares global scope, inline HTML attributes call globals
(e.g. `onerror="handleWatchPosterError(this)"`), the service worker hardcodes asset URLs, the
Android WebView mirrors every file, and cache-busting is manual `?v=NNN`. A rewrite touches all of
it at once, and the local preview can only prove the app still boots — not that 14k lines of
global-scope interdependency survived it (see "Verification" below).

## Non-negotiables

1. **Never commit `package-lock.json` or `deploy-vercel.ps1`.** If a rebase drags them in:
   `git stash push deploy-vercel.ps1 package-lock.json` → rebase → `git stash pop`.
2. **Bump the cache version on every asset change.** In `index.html` bump *every* `?v=NNN`
   (including `js/*.js` — forgetting these serves a stale helper against fresh `client.js` and
   throws `ReferenceError`), and bump `CACHE_NAME` in `service-worker.js`. Keep them in sync.
3. **Keep `android/app/src/main/assets/` in sync with the repo root.** These copies sometimes carry
   their own edits — diff before overwriting; prefer applying the same targeted edit to both.
4. **Deploys upload the working tree**, not just committed files. Uncommitted WIP can leak to
   production. Check `git status` before deploying.
5. **Supabase keys served to the browser must be the anon/publishable key — never `service_role`.**

## Conventions

- Line endings are **CRLF**. Scripted patches must match `\r\n` or they silently fail.
- Files are large; prefer targeted string replacement over rewriting a whole file.
- Commit messages end with:
  `Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>`
- Validate before committing: `node --check client.js` (and any edited `js/*.js`), plus a
  brace-balance check on `styles.css`. `npm run check` runs the project's own gate.

## Verification — important

`npm run dev` (`animetv-local.js`, port 4180) **works**. It boots in ~3 s and serves the whole
app, so the local preview *is* usable for verifying UI, routing, and rendering. It used to hang on
the initial catalog load; that is fixed. Ignore older notes claiming otherwise.

- **Static validation is necessary but NOT sufficient — use the browser.** `node --check` and grep
  pass clean on real, shipped breakage. Two cases from this repo: a regex rewrite that turned \b
  into literal backspace characters and silently disabled the adult classifier, and a `supabase`
  global colliding with the CDN's own global, which killed every login button. Both passed
  `npm run check`. Only running the app caught them.
- `npm run check` / `npm test` are the floor, not the gate. Two guards were added after those
  bugs: `scripts/check-global-collisions.mjs` (the `supabase` class of bug) and
  `scripts/check-asset-versions.mjs` (mixed `?v=` or a stale `CACHE_NAME`).
- **Bump `?v=NNN` before reloading the preview.** Otherwise you verify a cached copy and wrongly
  conclude the change "didn't apply". This has caused false negatives repeatedly.
- Harness caveat: programmatic `window.scrollTo` does not move the page here, so verify
  scroll-related CSS by reading computed styles, not by scripted scrolling.
- Real stream playback, scrapers, and OAuth login still realistically get verified **on the
  deployed site** after a push.
- Don't claim playback works without evidence; say what was and wasn't verified.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JSolanoDev/AnimeTV](https://github.com/JSolanoDev/AnimeTV) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
