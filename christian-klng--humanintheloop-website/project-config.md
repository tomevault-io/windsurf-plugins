---
trigger: always_on
description: Website for Human in the Loop. Plain HTML, CSS, and vanilla JavaScript served via nginx + Node.js API in a Docker container. A build-time Node.js script generates per-route HTML for OG meta tags; the API server regenerates them automatically after every admin change. An admin panel allows editing events and resources via the browser.
---

# Human in the Loop — Website

## Project Overview

Website for Human in the Loop. Plain HTML, CSS, and vanilla JavaScript served via nginx + Node.js API in a Docker container. A build-time Node.js script generates per-route HTML for OG meta tags; the API server regenerates them automatically after every admin change. An admin panel allows editing events and resources via the browser.

## Tech Stack

- HTML5, CSS3 (custom properties), vanilla JS
- Google Fonts (Switzer via Fontshare)
- Express.js API server (admin CRUD + data aggregation)
- Deployed via Coolify (Docker/nginx + Node.js) on https://humanintheloop.academy

## Project Structure

```
/
├── index.html              Main HTML template (with OG placeholders)
├── css/styles.css          All styles, design system variables
├── js/app.js               SPA router (History API), event rendering
├── js/admin.js             Admin panel UI logic
├── events/
│   └── events.json         Bundled event data (migration seed)
├── library/
│   └── resources.json      Bundled resource data (migration seed)
├── server/
│   ├── api.js              Express API server (auth + CRUD)
│   └── package.json        API server dependencies
├── scripts/
│   ├── generate-pages.js   Build-time OG meta tag generator
│   └── migrate-to-individual.js  Splits bundled JSON into individual files
├── nginx.conf              nginx routing + API proxy configuration
├── Dockerfile              Multi-stage: node build + nginx + API serve
├── docker-entrypoint.sh    Container startup (migration, OG gen, servers)
├── favicon.ico             Favicons and web manifest
├── site.webmanifest
├── corporate-design-system.md  Corporate Design System (colors, typography, print, etc.)
└── CLAUDE.md
```

## Corporate Design System

The file `corporate-design-system.md` contains the full Corporate Design System for Human in the Loop. It covers brand identity, logo usage, color system (HEX, RGB, CMYK), typography (Switzer), spacing, components, imagery guidelines, print specifications (business cards, flyers, ads, roll-ups), presentation templates, social media formats, and accessibility requirements. Refer to this document when creating any visual materials — digital or print.

## Naming

- **Never abbreviate "Human in the Loop"** — do not use "HITL" as it has unfortunate connotations in German. Always write the full name.

## Key Conventions

- **No CSS frameworks** — custom CSS with design tokens in `:root` variables
- **No inline styles** — all styling via classes in css/styles.css (exception: styleguide color swatches)
- **Events are data-driven** — individual JSON files on `/files/` volume, served via API
- **Library resources are data-driven** — individual JSON files on `/files/` volume, served via API
- **Media files served from `/files/` volume** — Docker volume mounted at `/files/`, referenced as `/files/library/...` in resource JSON
- **Path-based SPA routing** — URLs use `/`, `/events`, `/event/{id}`, `/library`, `/resource/{id}`, `/styleguide`, `/privacy`, `/terms`, `/imprint`, `/admin`
- **Semantic HTML** — use `<a>` and `<button>` (not `<div onclick>`), include ARIA labels
- **OG meta tags** — generated per-route at Docker build time via `scripts/generate-pages.js`; regenerated at container startup from volume data; also regenerated automatically by the API server after every admin write operation (create/update/delete); also updated client-side on navigation

## Color Palette

| Variable         | Hex       | Use                        |
|-----------------|-----------|----------------------------|
| `--accent`       | `#FFD166` | Primary buttons, accents   |
| `--secondary`    | `#073B4C` | Secondary buttons, bgs     |
| `--warning`      | `#EF476F` | Warnings, errors           |
| `--success`      | `#06D6A0` | Confirmations              |
| `--info`         | `#118AB2` | Notifications              |
| `--text-primary` | `#111111` | Main text                  |
| `--text-secondary`| `#8A8F98`| Muted text                 |

## Data Storage on `/files/` Volume

```
/files/
  ├── events/
  │   ├── event-slug.json           Individual event JSON files
  │   └── ...
  ├── library/
  │   ├── resource-slug/
  │   │   ├── resource.json         Resource data
  │   │   ├── thumb.jpg             Media files
  │   │   └── ...
  │   └── ...
  ├── uploads/                      Admin-uploaded media files
  │   ├── events/                 Event-related uploads
  │   ├── library/                Resource-related uploads
  │   └── ...
```

On first container startup, `migrate-to-individual.js` splits the bundled `events.json` and `resources.json` into individual files on the volume (idempotent).

## Admin Panel

- **Access**: Navigate to `/admin` (no link in public navigation)
- **Authentication**: Username + password login via `ADMIN_USER` and `ADMIN_PASSWORD` environment variables

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [christian-klng/humanintheloop_website](https://github.com/christian-klng/humanintheloop_website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
