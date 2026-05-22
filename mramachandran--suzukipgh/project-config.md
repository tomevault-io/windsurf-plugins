---
trigger: always_on
description: A static HTML/CSS/JS website for the Suzuki Association of Pittsburgh, a small nonprofit music education organization. The site is hosted on **GitHub Pages** with an **AI-powered admin interface** hosted on **Netlify** that lets non-technical admins update the site in plain English.
---

# Suzuki Association of Pittsburgh — Website

## What This Project Is

A static HTML/CSS/JS website for the Suzuki Association of Pittsburgh, a small nonprofit music education organization. The site is hosted on **GitHub Pages** with an **AI-powered admin interface** hosted on **Netlify** that lets non-technical admins update the site in plain English.

---

## Live URLs

| Purpose | URL |
|---|---|
| Live website | `https://mramachandran.github.io/suzukipgh/` |
| Admin interface | `https://<your-netlify-site>.netlify.app/admin` |
| GitHub repo | `https://github.com/mramachandran/suzukipgh` |

---

## How the System Works

```
Admin types a request  →  Netlify function (admin-ai.js)  →  Claude AI edits HTML
                                                                       ↓
        Site updates live  ←  GitHub Pages deploys  ←  Commit pushed to GitHub
```

1. Admin visits the `/admin` page on Netlify, enters the admin password
2. They type a plain-English request (e.g. "Add an event on June 5 at 2pm...")
3. The Netlify serverless function (`netlify/functions/admin-ai.js`) fetches the current HTML from GitHub, sends it to Claude with the request, and gets back a JSON response with the updated file content
4. The function commits the changes back to GitHub
5. GitHub Pages auto-deploys — site is live in ~2 minutes

---

## File Structure

```
suzukipgh/
├── index.html              # Homepage (testimonials, welcome, announcements)
├── about.html              # About the Suzuki Method
├── teachers.html           # Teachers directory with contact info
├── events.html             # Events calendar (most frequently updated)
├── board.html              # Board members
├── programs.html           # Programs offered
├── faq.html                # FAQ accordion
├── resources.html          # Downloadable guides and links for parents
├── contact.html            # Contact page with Google Maps
├── admin.html              # AI-powered admin interface (password protected)
├── 404.html                # Custom error page
├── styles.css              # Main stylesheet
├── css/styles.css          # Enhanced stylesheet
├── js/main.js              # Main JavaScript (nav, forms, back-to-top, etc.)
├── images/
│   ├── logo.jpg            # Organization logo
│   └── logo.svg            # SVG logo
├── favicon.svg             # Site favicon
├── sitemap.xml             # SEO sitemap
├── robots.txt              # Search engine directives
├── netlify.toml            # Netlify config (functions dir, redirects, headers)
├── netlify/
│   └── functions/
│       └── admin-ai.js     # Serverless function: AI + GitHub integration
└── docs/
    ├── README.md           # Overview and setup instructions
    ├── DEPLOYMENT.md       # GitHub Pages deployment steps
    ├── ADMIN-SETUP.md      # Admin interface one-time setup guide
    ├── UPDATING-EVENTS.md  # How to update events manually
    ├── GITHUB-PAGES-SETUP.md
    └── QUICK-START.md
```

---

## Tech Stack

- **Frontend**: Pure HTML5, CSS3, vanilla JavaScript — no frameworks, no build step
- **Fonts**: Crimson Pro (headings) + Work Sans (body) via Google Fonts
- **Hosting**: GitHub Pages (static site)
- **Admin backend**: Netlify Functions (Node.js serverless)
- **AI**: Claude API (`claude-haiku-4-5-20251001`) via Anthropic — Haiku is fast and more than capable for the surgical-op workload. Don't upgrade to Sonnet without a specific reason; Haiku keeps response times short.
- **Version control**: Git / GitHub (`mramachandran/suzukipgh`)

---

## Environment Variables (Netlify)

Set these in Netlify → Site configuration → Environment variables:

| Variable | Description |
|---|---|
| `ADMIN_PASSWORD` | Password for the `/admin` login screen |
| `ANTHROPIC_API_KEY` | Anthropic API key for Claude |
| `GITHUB_TOKEN` | GitHub Personal Access Token with `repo` scope |
| `GITHUB_REPO` | Repo in `owner/name` format — e.g. `mramachandran/suzukipgh` |

### Netlify config gotchas

- **Function timeout is NOT configurable via `netlify.toml`.** It's determined by the Netlify plan (≈10s Free, 26s Pro, up to 30s higher tiers). Do **not** add `timeout = 26` (or similar) under `[functions]` — it's invalid syntax and will fail every deploy at the config-parse stage, silently leaving the last successful deploy live. We've been bitten by this before.
- With the ops-based admin-ai rewrite, actual function time is ~5–8s, so no extension is needed.
- Keep the `[functions]` table minimal or absent. Valid keys are things like `node_bundler`, `external_node_modules`, `included_files` — not `timeout`.

### Custom domain + SSL

- `suzukipittsburgh.org` DNS points correctly to Netlify (A → 75.2.60.5), but the Let's Encrypt cert for the custom domain has not been provisioned yet — TLS requests get a `*.netlify.app` cert, which fails SAN validation in `curl`. Browsers mostly paper over this.
- Fix when ready: Netlify dashboard → Domain management → add/verify `suzukipittsburgh.org` → provision certificate.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mramachandran/suzukipgh](https://github.com/mramachandran/suzukipgh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
