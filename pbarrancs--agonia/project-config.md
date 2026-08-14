---
trigger: always_on
description: Brand presence website for **agonia**, an independent climbing apparel collective
---

# CLAUDE.md — agonia website

## What this project is

Brand presence website for **agonia**, an independent climbing apparel collective
based in Mexico. The site communicates the brand's identity, mission, and values
to the climbing and outdoor community. There is no e-commerce or purchasing flow —
the catalog is display-only.

## Latest Updates (2026-07-28)

- Inventory availability now comes from the workbook itself: missing values are normalized to `0`, and `Stock` is derived as `Unidades - Vendidas`.
- The catalog sync logic reads `inventario_agonia.xlsx` as the source of truth, so the site can use the Excel file directly to mark products as available or sold out.
- The old `fix_inventario.py` step is no longer required; its normalization logic now runs inside the existing sync scripts.
- Catalog card headings use UI-level display-name mapping (no manual edits to `productos.csv`).
- Catalog section titles use collection display names only: `Fall T`, `Cadenas`, `Back Logo`, `Lágrima`.
- Home page media section label is `Clips` (previously `Videos`).
- Home hero keeps only one CTA card (`Ver catálogo`); the extra `Nosotros` card was removed.
- Entry-point behavior reverted: `/` loads the home page directly (no Firebase redirect to `/catalogo`), and `/catalogo` remains available via navigation.
- Nosotros page now includes updated manifesto copy and a centered Perrito illustration block at the bottom.
- Current Perrito image source:
  `/assets/images/Ilustraciones/Perrito_03/P3_PNG_/P3_BlancoNegro_png/Perrito3_blanco_FondoOscuro.png`.

---

## Brand context

- **Category:** Independent climbing apparel (streetwear inspired by climbing culture)
- **Location:** Mexico
- **Audience:** Climbing and outdoor community in Mexico and Latin America
- **Tone:** Independent, aesthetic, community-driven
- **Social:** Instagram [@\_\_\_agonia](https://www.instagram.com/___agonia/) · YouTube [@agon_ia](https://www.youtube.com/@agon_ia)
- **Catalog rule:** Display only — no shopping cart, no checkout, no purchasing flow

---

## Current stack

| Tool                 | Version / Source                             | Role                                                     |
| -------------------- | -------------------------------------------- | -------------------------------------------------------- |
| **Astro**            | 4.16.x (`^4.16.0`)                           | Static site framework — builds to `new-astro-site/dist/` |
| **CSS**              | Plain CSS (no framework)                     | Layout, typography, brand colors                         |
| **Google Fonts**     | `New Rocker`, `Nova Cut`, `Syne Mono`        | Brand typography (loaded via CDN)                        |
| **Font Awesome 5**   | Bundled locally in `public/assets/webfonts/` | Social and UI icons                                      |
| **Vanilla JS**       | Inline `<script>` blocks in Astro components | Catalog interactivity (color/size filtering)             |
| **Firebase Hosting** | Project:`agonia-255fe`                       | Static site deployment and CDN                           |

No React, no Vue, no Svelte. No Tailwind, no PostCSS. No jQuery.

---

## Brand design tokens

```css
/* Defined in new-astro-site/src/styles/global.css */
--color-titulo: #e8431e; /* New Rocker — headings */
--color-subtitulo: #f4ec62; /* Nova Cut — subheadings */
--color-cuerpo: #fcfcfc; /* Syne Mono — body text */
--color-bg: #1a1a1a; /* Dark background */
```

---

## Current project structure

```
agonia/                                   ← repo root
├── firebase.json                         ← Firebase Hosting config; serves from new-astro-site/dist/
├── .firebaserc                           ← Firebase project alias: agonia-255fe (never modify)
├── .gitignore
├── README.md
├── CLAUDE.md                             ← this file
├── new-astro-site/                       ← Astro project (the live site)
│   ├── package.json                      ← astro ^4.16.0; scripts: dev / build / preview
│   ├── astro.config.mjs                  ← output: 'static'
│   ├── dist/                             ← build output; what Firebase deploys (gitignored)
│   ├── public/                           ← static assets copied as-is into dist/
│   │   └── assets/
│   │       ├── a_icono.ico               ← browser favicon
│   │       ├── css/
│   │       │   └── fontawesome-all.min.css
│   │       ├── images/
│   │       │   ├── bg.jpg                ← page background
│   │       │   ├── avatar.jpg / avatar_2.jpg
│   │       │   ├── nico_razo.gif / .mp4
│   │       │   ├── Fondos/               ← 7 background color variants
│   │       │   ├── fulls/ / thumbs/      ← gallery images (06 each)
│   │       │   ├── Logos/                ← Logo_A_lagrima, Logo_figura, Logo_horizontal (8 variants each)
│   │       │   ├── Ilustraciones/        ← Perrito_01–08, Personaje_A
│   │       │   └── playeras/             ← 29 product photos (PNG) + placeholder.svg
│   │       └── webfonts/                 ← 15 Font Awesome font files
│   └── src/
│       ├── env.d.ts
│       ├── data/
│       │   └── productos.csv             ← 144 rows; source of truth for catalog
│       ├── layouts/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pbarrancs/agonia](https://github.com/pbarrancs/agonia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
