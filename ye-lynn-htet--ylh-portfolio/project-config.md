---
trigger: always_on
description: Personal portfolio for a senior mobile developer (iOS + Flutter) based in Kobe, Japan. Single-page site with six sections: Header, Hero, Skills, Experience, Projects, Contact.
---

# Ye Lynn Htet — Portfolio

Personal portfolio for a senior mobile developer (iOS + Flutter) based in Kobe, Japan. Single-page site with six sections: Header, Hero, Skills, Experience, Projects, Contact.

## Tech Stack

| Layer | Choice | Version |
|-------|--------|---------|
| Framework | Next.js (App Router) | 16.2.9 |
| UI | React | 19.2.4 |
| Styling | Tailwind CSS | v4 (no tailwind.config) |
| Language | TypeScript | 5.x |
| Fonts | Geist Sans + Geist Mono (next/font/google) | — |
| Lint | ESLint with eslint-config-next | 9.x |

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

## Architecture

- **App Router only** — no Pages Router. All routes live under `src/app/`.
- **Single-page layout** — sections scroll on the same route (`/`). Navigation uses anchor IDs (`#skills`, `#experience`, `#projects`, `#contact`).
- **Server Components by default** — add `"use client"` only when you need interactivity (state, effects, event handlers). The site is fully static.
- **No API routes yet** — all data lives in `const` arrays and objects at the top of `page.tsx`. No database, no fetch. Future API binding is planned; see `instructions.md` for the full preparation guide.
- **Detail screens planned** — each section will have a dedicated `/skills`, `/experience`, `/projects` route. Individual projects get `/projects/[slug]` detail pages.
- **Home screen: 3-item preview** — the landing page shows 3 items per section with an "Explore More →" button linking to the full detail route. This keeps the home page scannable while offering depth.
- **Mobile-first responsive** — all Tailwind classes start at the smallest breakpoint and scale up (`sm:`, `md:`, `lg:`).

## Design System

### Colors
| Token | Class | Hex | Usage |
|-------|-------|-----|-------|
| Background | `bg-slate-950` | #020617 | Full-page canvas |
| Card surface | `bg-slate-900/60` | — | Cards, skill groups, contact footer |
| Body text | `text-slate-400` | #94a3b8 | Descriptions, secondary |
| Headings | `text-slate-50` | #f8fafc | h1, h2, h3 titles |
| Primary accent | `indigo-400/500` | #6366f1 / #818cf8 | Hero, section labels, CTA, header |
| iOS accent | `sky-400/500` | #0ea5e9 | Swift/UIKit/SwiftUI groups, SPOTV project |
| Growth accent | `emerald-400/500` | #10b981 | Tools group, Binary Lab, Rehab System |
| Warm accent | `amber-400/500` | #f59e0b | Flutter/Dart, Kakely, Light Idea |
| Streaming accent | `violet-400/500` | #8b5cf6 | SAYA project |
| POS accent | `rose-400/500` | #f43f5e | Flash Mall project |
| Borders | `border-slate-700/800` | — | Cards, tags, sections, buttons |
| Muted labels | `text-slate-500/600` | — | Section eyebrows, label text |
| Code-line decor | `h-px bg-indigo-500/40` | — | `<SectionLabel>` left decoration |

### Typography
- **Display/headings**: Geist Sans (`--font-geist-sans`), bold, tight tracking
- **Body**: Geist Sans, regular weight, relaxed leading
- **Utility/mono**: Geist Mono (`--font-geist-mono`), uppercase tracking-widest, used for section eyebrow labels and logo label text

### Accent token system
All accents are defined in the `accentTokens` record at the top of `page.tsx`. Each accent has three slots:

```ts
type Accent = "indigo" | "sky" | "emerald" | "amber" | "violet" | "rose";

const accentTokens: Record<Accent, { bar: string; tag: string; glow: string }> = {
  indigo:  { bar: "bg-indigo-500",  tag: "text-indigo-400  border-indigo-500/20",  glow: "bg-indigo-500/10" },
  // ...etc
};
```

To add a new accent: add an entry to `Accent` union + `accentTokens`. Tag classes use `border-<color>-500/20` for a muted border that matches the accent.

### Reusable micro-components (all in `page.tsx`)
- `<SectionLabel>` — mono eyebrow label with indigo code-line decoration (`h-px w-8`)
- `<AccentBar accent>` — small colored bar (`h-1 w-8 rounded-full`)
- `<Tag accent>` — rounded pill badge with accent border + text + `bg-slate-900/80`
- `<HeroIllustration />` — orbiting logo ring (8 tech logos on circular track with slow rotation, speeds up on hover)

### Hero illustration: logo orbit
8 technology logos arranged on a circular orbit track. The outer ring rotates clockwise via `animate-orbit` (24s), each logo icon counter-rotates via `animate-unorbit` to stay upright. On `group-hover` both switch to `-fast` variants (8s). Logos use semi-transparent fills to blend with the background. Three concentric ghost rings provide depth. Logos positioned using pre-computed `translate(cos(angle)*R, sin(angle)*R)` offsets on a 112px radius.

### Section patterns
| Section | Pattern | Key classes |
|---------|---------|-------------|
| Header | Sticky nav with backdrop-blur | `sticky top-0 z-50 border-b border-slate-800/50 bg-slate-950/80 backdrop-blur-md` |
| Hero | Two-column (text left, orbit right) | `flex flex-col sm:flex-row gap-10 sm:gap-16` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ye-lynn-htet/ylh-portfolio](https://github.com/ye-lynn-htet/ylh-portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
