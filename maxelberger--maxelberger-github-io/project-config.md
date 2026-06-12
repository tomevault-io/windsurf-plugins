---
trigger: always_on
description: ﻿# GitHub Copilot Instructions for MaxeLBerger.github.io
---

﻿# GitHub Copilot Instructions for MaxeLBerger.github.io

## Overview

Personal portfolio site for Maximilian Haak — fullstack web developer & AI specialist from Bruckmühl/Rosenheim. Static site (HTML + CSS + vanilla JavaScript), deployed to GitHub Pages.

**Live:** [maximilianhaak.de](https://maximilianhaak.de)

## Architecture

Plain static site — **no build step**, no submodules, no frameworks.

```
.
├── index.html              # Homepage (hero slider + sections)
├── impressum.html          # Legal info
├── datenschutz.html        # Privacy policy
├── assets/
│   ├── css/
│   │   └── main.css        # Stylesheet (~2700 lines, design tokens + theming)
│   ├── js/
│   │   └── main.js         # Slider, theme, i18n, animations (~1340 lines)
│   └── img/                # Backgrounds, favicons, icons, profile, projects, screenshots
├── projects/               # Project detail pages
│   ├── main.css            # Project-page-specific styles
│   ├── aicaptain.html
│   ├── coha.html
│   ├── e46-studio.html
│   ├── imkerei-feuerstein.html
│   ├── shookroko.html
│   └── soundoflvke.html
├── docs/                   # architecture / development / deployment guides
├── tools/                  # Local dev tooling (NOT deployed)
│   └── mcp-portfolio-server/   # Local MCP dev tool
└── .github/workflows/
    └── deploy.yml          # Single-job deploy to GitHub Pages
```

## Tech Stack

- **HTML5 / CSS3 / vanilla JavaScript (ES6+)** — no React, no Vue, no bundler
- **GSAP 3.12 + ScrollTrigger** via CDN for animations
- **Inter** (Google Fonts) — loaded after cookie consent (GDPR)
- **i18n**: custom DE/EN dictionary in [assets/js/main.js](../assets/js/main.js) (DE is default)
- **Hosting**: GitHub Pages with custom domain (`CNAME` → `maximilianhaak.de`)

## Core Architectural Concepts

### Homepage hero

The top `#hero` uses the E46 photo as the single full-bleed background. Keep the responsive
`assets/img/backgrounds/hero-e46-640.webp`, `hero-e46-960.webp`, `hero-e46-1440.webp`, and full
`hero-e46.webp` variants wired through the `.hero-bg-squared` `<picture>` sources; the active image is the LCP element,
so keep it eager with `fetchpriority="high"` and do not add competing CSS-background preloads;
do not switch back to the older `herosquared` hero. Keep the dedicated `.hero-section--square` and
`.hero-bg-squared` crop/veil rules. Place the hero text in the right-side grid column (`.hero-section--single`) on
wide screens and use the darker right-side/bottom contrast veil for readability over the E46 photo.
Hero credibility badges belong inside `.hero-text` below the headline as normal `.hero-badges` flex content. Keep them
as standalone white SVG icons with short text labels and without pill/card backgrounds; do not position them as absolute
overlays over the photo. Keep the `Haak.` word in the hero headline as themed `.gradient-text` using the active color
palette, with only a subtle drop-shadow for photo contrast. Animate these `.hero-badge` items as part of the homepage
hero entrance sequence.

### About section

`#about` sits directly after `#projects` and before the tech stack. Keep it as a personal editorial section with an
E46 media frame, short biographical copy, and compact facts. The current media asset is the optimized
`assets/img/profile/uebermich.webp` (portrait, 1000×1334, keep under ~300 KB) in a `.about-image-frame`; preserve the
lower focus so the car/person stay visible. Keep this image large and cleanly cropped in a restrained 5:6 media frame
with subtle border/shadow and no visible caption. The copy column should stay intentionally narrow for comfortable
reading (about 54ch max), rather than stretching across the available grid width. Use
`Über mich` / `About me` as the real section heading, not as an eyebrow; section-note/hint callouts belong only to
`#projects`. If this later becomes video, use
`assets/video/e46-about.mp4` with a poster fallback. All visible copy still goes through `data-i18n` keys in
`assets/js/main.js`; translated image alt text uses `data-i18n-alt`.

### Hero project slider

`#projects` contains exactly ONE `.hero-slides-container` dedicated to own projects: E46 Studio, AI Captain,
Medieval TD, Shookroko, and dog-kennel-online. Above the slider sits a two-option `.project-mode-selector`
segmented pill with
`Kundenprojekte` (left) and `Eigene Projekte` (right). `Eigene Projekte` is the active, enabled option and controls
the single slider. `Kundenprojekte` is intentionally locked (`disabled`, `aria-disabled="true"`, `tabindex="-1"`,
`.is-locked` modifier with a small lock SVG and a visually-hidden `.project-mode-status`) until paid customer work
exists; do NOT wire it up or render customer slides on the homepage. The customer i18n keys (`slide.imkerei.*`,
`slide.coha.*`, `slide.soundoflvke.*`, `slide.danielbrecheis.*`, `slide.kayaseeds.*`, `slide.jkentertainment.*`) are
intentionally retained in [assets/js/main.js](../assets/js/main.js) for future use.

The `ProjectSlider` class in [assets/js/main.js](../assets/js/main.js) handles:

- GSAP-powered transitions (with CSS fallback)
- Touch/swipe + keyboard navigation
- Per-slide theme switching via `data-theme` → `data-project-theme` on `<html>`
- Container height calculation (slides have varying heights)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MaxeLBerger/MaxeLBerger.github.io](https://github.com/MaxeLBerger/MaxeLBerger.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->
