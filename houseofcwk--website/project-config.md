---
trigger: always_on
description: - **Product:** CWK. Experience — PLOS Product Website
---

# AGENTS.md — CWK PLOS Website

## Project Identity

- **Product:** CWK. Experience — PLOS Product Website
- **Type:** Static marketing/product site
- **Tagline:** "The sports agent for entrepreneurs."
- **Philosophy:** Build. Learn. Earn. Play.
- **Target URL:** cwkexperience.com (Cloudflare Pages — project: houseofcwk)
- **Reference site:** cwkexperience.com (old site on page builder — used as content/design source only, not the deploy target)

## Tech Stack

| Layer         | Technology                                |
|---------------|-------------------------------------------|
| Framework     | Astro (latest stable)                     |
| Language      | TypeScript (strict)                       |
| Styling       | Scoped CSS + global CSS variables         |
| Content       | Astro Content Collections (Markdown/MDX)  |
| Islands       | React (for interactive components only)   |
| Deployment    | Cloudflare Pages                          |
| Analytics     | Cloudflare Web Analytics                  |
| Fonts         | DM Sans (primary, all elements), Agrandir (display accents only) |

## Architecture Rules

- **Static-first** — every page should be statically generated at build time. No SSR unless explicitly required.
- **Islands architecture** — interactive components (quiz, forms) use Astro's `client:*` directives. Default to `client:visible` for lazy hydration.
- **Content Collections** — all case studies, portfolio items, and articles live in `src/content/` as Markdown/MDX with typed frontmatter schemas.
- **No JavaScript by default** — pages ship zero JS unless an island component is present. This is Astro's default; preserve it.
- **Scoped styles** — use `<style>` blocks in `.astro` files for component-scoped CSS. Global styles live in `src/styles/global.css`.
- **SEO-first** — every page must have: `<title>`, `<meta name="description">`, Open Graph tags, canonical URL. Use a shared `<SEO>` component.
- **Responsive** — mobile-first design. Breakpoints: 640px (sm), 768px (md), 1024px (lg), 1280px (xl).

## File Conventions

```
cwk-plos-site/
├── astro.config.mjs           # Astro config (Cloudflare adapter, integrations)
├── package.json
├── tsconfig.json
├── public/                    # Static assets (favicons, robots.txt, images)
│   ├── favicon.svg
│   ├── robots.txt
│   ├── og-image.png           # Default Open Graph image
│   └── images/                # Optimized images
├── src/
│   ├── layouts/               # Page layouts
│   │   ├── Base.astro         # HTML shell: <head>, fonts, analytics, footer
│   │   └── Article.astro      # Layout for case study / article pages
│   ├── components/            # Reusable UI components
│   │   ├── Header.astro       # Global navigation
│   │   ├── Footer.astro       # Global footer
│   │   ├── Hero.astro         # Homepage hero section
│   │   ├── ProductHero.astro  # Product page hero
│   │   ├── FeatureShowcase.astro # Alternating text + mockup layout
│   │   ├── PillarCard.astro   # Mind/Body/Soul/Pocket card
│   │   ├── CaseStudyCard.astro
│   │   ├── SEO.astro          # Meta tags component
│   │   ├── BrandMirror.tsx    # Interactive quiz (React island)
│   │   ├── WaitlistForm.tsx   # Email capture form (React island)
│   │   └── mocks/             # Static product mockup components
│   │       ├── MockDashboard.astro
│   │       ├── MockRelationships.astro
│   │       ├── MockPipeline.astro
│   │       ├── MockActions.astro
│   │       ├── MockKaia.astro
│   │       └── MockDestination.astro
│   ├── pages/                 # File-based routing
│   │   ├── index.astro        # Homepage (with waitlist + product preview)
│   │   ├── about.astro        # About Kris
│   │   ├── work.astro         # Portfolio grid
│   │   ├── work/[slug].astro  # Individual case study pages
│   │   ├── product.astro      # Agent+ prototype mockup intro page
│   │   ├── brand-mirror.astro # Brand Mirror quiz page
│   │   ├── side-quests.astro  # Side quests page
│   │   └── 404.astro          # Custom 404
│   ├── content/               # Content Collections
│   │   ├── config.ts          # Collection schemas
│   │   └── work/              # Case study markdown files
│   │       ├── the-lab-miami.md
│   │       ├── rob-dial.md
│   │       ├── spraycation.md
│   │       ├── dawa.md
│   │       ├── stephy-lee.md
│   │       ├── raasin-in-the-sun.md
│   │       ├── pay-the-creators.md
│   │       └── lifes-tapestry.md
│   └── styles/
│       └── global.css         # CSS variables, resets, typography, brand system
└── docs/                      # Project documentation (not deployed)
```

## Brand Design System

### CSS Variables (Official Brand Palette)

> See `docs/DESIGN.md` for full design system reference.

```css
:root {
  /* Primary Palette */
  --bg:     #07090F;  /* Page background */
  --bg2:    #0B0E18;  /* Cards, panels, elevated surfaces */
  --bg3:    #101422;  /* Tab panels, deep nested elements */
  --text:   #EEF0FF;  /* ALL headings, body, labels */
  --cyan:   #00E5FF;  /* Primary accent: CTAs, mission bars */
  --purple: #7B61FF;  /* Secondary accent: gradients, PLOS */
  --pink:   #FB3079;  /* Tertiary accent: eyebrow labels, warnings */

  /* Transparency */
  --glass:  rgba(255, 255, 255, 0.03);  /* Card fills */

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [houseofcwk/website](https://github.com/houseofcwk/website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
