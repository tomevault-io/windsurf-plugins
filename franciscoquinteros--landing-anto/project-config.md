---
trigger: always_on
description: Personal link-in-bio landing page for Anto Lancuba (antonellalancuba.com).
---

# landing-anto

Personal link-in-bio landing page for Anto Lancuba (antonellalancuba.com).

## Tech stack

- Vanilla HTML/CSS/JS — no build step, no framework
- Netlify Functions v2 (ES Modules, `.mjs` files)
- Netlify Blobs for persistent data storage
- GitHub API (@octokit/rest) for committing data back to repo
- Content is in Spanish

## Project structure

```
index.html / style.css        — Public landing page
dashboard/                     — Admin panel (password-protected)
netlify/functions/             — Serverless functions (.mjs, Functions v2)
data/site-data.json            — Static fallback for site content
netlify.toml                   — Netlify config and redirects
.github/workflows/deploy.yml   — Auto-deploy on push to master
```

## Serverless functions

| Function | Purpose |
|---|---|
| `auth.mjs` | Login — validates ADMIN_PASSWORD |
| `site-data.mjs` | GET site content (Blobs → fallback to static JSON) |
| `save-data.mjs` | POST updated content (Blobs + GitHub commit) |
| `track.mjs` | POST link click analytics |
| `metrics.mjs` | GET analytics data |
| `upload-image.mjs` | POST profile image upload |
| `upload-link-image.mjs` | POST link thumbnail upload |

## Commands

- Dev server: `npm run dev` (runs `netlify dev`)
- Deploy: `npm run deploy` (runs `netlify deploy --prod`)
- Auto-deploys on push to `master` via GitHub Action

## Environment variables

Required in Netlify UI and locally in `.env`:
- `ADMIN_PASSWORD` — dashboard login
- `GITHUB_TOKEN` — for committing data/images back to repo
- `GITHUB_REPO` — format: `owner/repo`

## Gotchas

- **No `fs`/`path` in Functions v2**: Netlify Functions v2 run in Deno-like edge environment. Importing `fs` or `path` crashes the function. Use `fetch()` or Netlify Blobs instead.
- **Blobs vs static fallback**: `site-data.mjs` tries Blobs first, falls back to reading `data/site-data.json` via fetch. After saving from dashboard, Blobs has the latest data but the static file is only updated via GitHub commit (async).
- **Duplicated `verifyAuth`**: Auth verification is copy-pasted in `save-data.mjs`, `upload-image.mjs`, `upload-link-image.mjs`, and `metrics.mjs`. Changes must be applied to all four.
- **GitHub API SHA requirement**: When committing file updates via Octokit, you must first GET the current file to obtain its SHA, then PUT with that SHA.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/franciscoquinteros) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
