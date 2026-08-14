---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## ⚠️ Live site comes first — read this every session

This site is **in production**. Every push to `main` deploys to both
- Vercel: `https://www.meihodo.com` (custom domain; `meihodo.com` apex redirects to `www`)
- GitHub Pages: `https://meihodoliving.github.io/test/`

**The deployed site must stay rendering correctly at all times.** A broken deploy is the most expensive kind of failure here, because real visitors see it. Optimize every change around this priority.

### Before acting on any prompt

1. **Read the user's prompt twice.** If the request is bulk/mechanical — "delete X", "rename Y", "remove all Z", "rewrite paths", "consolidate", "move everything to …" — pause and ask yourself: *does the thing I'm about to remove or change exist for a reason that local testing won't surface?* This repo has at least three foot-guns that look harmless locally and break only on Vercel (see "Deployment invariants" below). When in doubt, **ask the user** before making the change. A 10-second clarification is cheaper than a broken production site.
2. **Pre-flight audit** before any bulk edit to HTML, CSS, or directory structure:
   ```bash
   grep -rn '/public/' --include='*.html' .                       # must be empty
   grep -rn 'href="\.\./[a-z]' --include='*.html' */experiences/   # must be empty
   [ -d public ] && echo BAD || echo OK                            # must say OK
   ```
3. **Never** run destructive commands (`rm -rf`, `git rm`, mass `sed`) without first proving the targets are safe to remove. Move-then-verify-then-delete, not delete-then-hope.

### Before pushing to main

1. `git diff --stat origin/main..HEAD` — eyeball the file count and which trees are touched. If a change you didn't expect appears, investigate before pushing.
2. Re-run the pre-flight audits above. They must still be clean after your edits.
3. If the change touches HTML structure or assets, push and then **curl-verify the live site** before walking away:
   ```bash
   curl -sL https://www.meihodo.com/en/experiences/samurai | grep -c 'class="experience-hero"'    # > 0
   curl -sI https://www.meihodo.com/en/experiences/components.css | head -1                       # 200, not 404
   curl -sI https://www.meihodo.com/styles.css | grep -i content-length                           # 156000+ bytes
   ```
4. If GitHub Pages goes red, fix it immediately. Check with `GH_TOKEN=<token> gh run list --repo meihodoliving/test --limit 3`. The most common failure is a deprecated `actions/*` version.

### Why local "works" ≠ deployed works

Local servers (`live-server`, `python http.server`) do NOT replicate Vercel's behavior:
- They don't auto-treat `public/` as a static-asset root.
- They don't strip trailing slashes (Vercel's `cleanUrls: true` does).
- They serve every file in the working tree without honoring `.vercelignore`.

A page can render perfectly locally and be completely broken on Vercel. **Trust the deployed-site curl checks, not local rendering, when verifying a fix is real.**

## Project

Static multilingual marketing site for **鳴鳳堂 (Meihodo) / Aso Cultural Resort**. Pure HTML + CSS + a tiny vanilla JS file ([lang-switcher.js](lang-switcher.js)). No build step for the live site — files are served as-is.

A separate Next.js 15 / React 19 / Tailwind 4 rewrite lives under [meihodo-rebuild/](meihodo-rebuild/). It is **excluded from the deployed site** (see [.vercelignore](.vercelignore)) and has its own toolchain — treat it as an independent project.

## Common commands

Run from repo root unless noted.

```bash
npm start                      # live-server on :3000 with reload (serves /index.html — the Japanese root)
npm run serve                  # python3 http.server on :8000 (no reload)

npm run lint                   # htmlhint index.html + stylelint styles.css (root files only)

npm run optimize:images:dry    # preview JPG/PNG → WebP conversions
npm run optimize:images        # convert + rewrite references across the repo + delete originals
```

For the Next.js rebuild:

```bash
cd meihodo-rebuild && npm run dev      # next dev --turbopack
cd meihodo-rebuild && npm run lint     # eslint
```

There are no tests in this repo.

## Architecture: the multilingual mirror

The site is published at four language roots that share an identical directory tree:

```
/index.html        ← Japanese root (also reachable as /ja/index.html)
/ja/  /en/  /zh-cn/  /zh-tw/
```

**`/ja/` is the source of truth.** `/en/`, `/zh-cn/`, `/zh-tw/` are structural mirrors of `/ja/` — same file paths, same HTML structure, only text/links translated. When adding a page, add it under `/ja/` first and propagate.

Each language tree contains the same set of sections: `about/`, `access/`, `accommodations/`, `bunshinkan/`, `edokan/`, `experiences/`, `geihinkan/`, `hinokinoma/`, `korokan/`, `location/`, `restaurant/`, `seiseikan/`, `stay/`. Sibling pages exist for `restaurant.html` at the language root.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [meihodoliving/test](https://github.com/meihodoliving/test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
