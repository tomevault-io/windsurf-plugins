---
trigger: always_on
description: Guidance for AI coding agents (and humans skimming for the rules) working in the
---

# AGENTS.md

Guidance for AI coding agents (and humans skimming for the rules) working in the
Pod repository. This is the canonical instruction file.

Pod is an open-source, MIT-licensed Ghost theme for podcasters, released by
[Magic Pages](https://www.magicpages.co). It turns any Ghost 5.93+ site into a
full podcast home: hero episode with a built-in audio player, iTunes-spec +
Podcasting 2.0 RSS feed at `/podcast/rss/`, a `/subscribe/` landing page that
deep-links to 16 podcast apps, and localised UI in six languages.

Live demo: [pod.magicpages.co](https://pod.magicpages.co).
Docs surface: the README, CONTRIBUTING, and CHANGELOG at the repo root.

This is a working, released codebase. The notes below describe what's actually
here.

## Setup & build

- **npm.** Install with `npm install`.
- `npm run dev` — Vite in watch mode; rebuilds `assets/built/` on change.
  Pair with a local Ghost bind-mounted at `content/themes/pod/` (bind-mount, not
  upload — see below).
- `npm run build` — one-off production build. Regenerates `assets/built/`.
- `npm run validate` — `gscan` against Ghost 6.x. Should report zero warnings.
- `npm run zip` — package a Ghost-ready `pod.zip` at the repo root (`scripts/zip.mjs`).
- `npm run changeset` — add a changeset for the current work.
- `npm run version` — bump `package.json` + regenerate `CHANGELOG.md` from
  accumulated changesets. Only ever run by CI on the Version PR branch.
- `npm run release` — build + tag + create the GitHub Release with `pod.zip`
  attached (`scripts/release.mjs`). Only ever run by CI when the Version PR
  merges.

## Tech stack

- **Handlebars** — Ghost's templating engine. No JSX, no framework runtime.
- **Tailwind CSS** — utility-first styling, `assets/css/main.css` is the entry.
  Real components extracted to CSS classes (`.pill`, `.chip`, `.pod-player`,
  `.cover-art`, …) where they repeat.
- **Vanilla JavaScript** — no framework. `assets/js/main.js` is the entry;
  `assets/js/player.js` is the audio player (~250 lines, no deps).
- **Vite** for the asset build (library mode isn't used; just the CSS +
  woff2 pipeline + JS bundling).
- **gscan** as the schema/API checker against a specific Ghost version.
- **Changesets** for versioning + CHANGELOG generation.
- **CSS custom properties** on `:root` for theme tokens — every colour flows
  from `--color-*`. Publisher's `@site.accent_color` sets the accent for the
  whole site. Never hard-code brand colours.

## Architecture

```
pod/
├── assets/
│   ├── css/main.css          # Tailwind entry + extracted components
│   ├── js/main.js            # Player, color-scheme toggle, pod:meta hydration
│   ├── js/player.js          # Audio player: skip ±30s, speed, waveform, keys
│   ├── fonts/                # Self-hosted woff2 sources
│   ├── img/default-cover.jpg # 3000×3000 RSS fallback (Ghost's stock cover
│   │                         # cropped square + re-compressed, ~360 KB)
│   └── built/                # Vite output (gitignored — regenerated on build)
├── locales/
│   └── en.json + de/fr/es/uk/it.json
├── partials/                 # Reusable Handlebars blocks
│   ├── audio-player.hbs
│   ├── cover-art.hbs         # Cover tile — <img> when feature_image, gradient
│   │                         # fallback + waveform + episode number otherwise
│   ├── header.hbs / footer.hbs / navigation.hbs
│   ├── post-card.hbs         # Archive list entry
│   ├── pod-duration.hbs      # Duration read from pod:duration marker
│   ├── subscribe-band.hbs    # Home + archive "listen on" band
│   ├── subscribe-pills.hbs   # Top-5 pills + more-link (shared surface)
│   └── subscribe-grid.hbs    # Full 11-15 platform grid on /subscribe/
├── podcast/
│   └── rss.hbs               # iTunes-spec + Podcasting 2.0 feed
├── subscribe.hbs             # /subscribe/ landing page
├── default.hbs               # Base layout, imports partials/header + footer
├── index.hbs                 # Home + tag + archive collections
├── post.hbs                  # Single episode
├── page.hbs / tag.hbs / author.hbs / error*.hbs
├── routes.yaml               # Ghost defaults + /podcast/rss/ + /subscribe/
├── screenshot-{desktop,mobile}.png  # Marketplace + README hero screenshots
├── docs/                     # README-only supplementary screenshots
├── scripts/
│   ├── zip.mjs               # Repo-root → pod.zip staging + package
│   └── release.mjs           # Tag + GitHub Release + pod.zip attach
├── .changeset/               # Changeset markdown files + config
└── .github/workflows/
    ├── ci.yml                # gscan + build + zip on push/PR
    ├── release.yml           # Version PR flow + release on merge
    └── deploy.yml            # release.published → deploy to pod.magicpages.co
```

## The Ghost integration contract

Pod is a **plain Ghost theme** — no admin patches, no server config changes.
Ghost 5.93+ loads it via **Settings → Design → Change theme → Upload theme**.

**Routes** — `routes.yaml` at the repo root is a complete ready-to-upload file:
Ghost's default routes (root collection + tag/author taxonomies) plus the two
entries Pod needs — `/podcast/rss/` (renders `podcast/rss.hbs` with
`content_type: text/xml`) and `/subscribe/` (renders `subscribe.hbs`). Publishers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [magicpages/pod](https://github.com/magicpages/pod) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
