---
trigger: always_on
description: > This file is the single source of truth for Claude Code working on this project.
---

# CLAUDE.md — Portfolio Project

> This file is the single source of truth for Claude Code working on this project.
> Read this before touching any file. Follow every rule here strictly.

---

## Project Overview

Personal portfolio website for **Muhammad Jamil Raza Attari**, an AI/ML Engineer,
Data Scientist, and Educator based in Karachi, Pakistan.

**Primary goals:**
- Attract international remote AI/ML job opportunities
- Convert freelance clients (Upwork / Fiverr)
- Build personal brand as an AI practitioner and educator

**Core identity tagline:**
> "I Build Intelligent AI Systems — and Teach Others to Do the Same"

**Audiences (in priority order):**
1. International remote recruiters (AI/ML roles)
2. Local Pakistani companies (job search)
3. Freelance clients (Upwork / Fiverr)

---

## Tech Stack & Tools

| Layer        | Technology                          | Version     |
|--------------|-------------------------------------|-------------|
| Framework    | Next.js (App Router)                | 14.x latest |
| Language     | TypeScript                          | 5.x         |
| Styling      | Tailwind CSS                        | 3.x         |
| Animations   | Framer Motion                       | 11.x        |
| Icons        | Lucide React                        | latest      |
| Fonts        | next/font/google (self-hosted)      | built-in    |
| Contact Form | Formspree                           | free tier   |
| Deployment   | Vercel                              | free tier   |
| Package Mgr  | npm                                 | —           |

**Font pairs (dual-mode):**
- Light mode: `Lora` (heading) + `Outfit` (body)
- Dark mode:  `JetBrains Mono` (heading) + `DM Sans` (body)

---

## Project Structure

```
jamil-portfolio/
├── app/
│   ├── layout.tsx          ← Root layout: font variables, metadata, ThemeProvider
│   ├── page.tsx            ← Single page: all sections composed here
│   ├── globals.css         ← CSS variables, Tailwind base, dark mode overrides
│   └── fonts.ts            ← next/font declarations for all 4 fonts
│
├── components/
│   ├── layout/
│   │   ├── Navbar.tsx      ← Logo, nav links, ThemeToggle
│   │   └── Footer.tsx      ← Social links, copyright
│   ├── sections/
│   │   ├── Hero.tsx        ← Name, tagline, CTAs, profile photo
│   │   ├── About.tsx       ← Value proposition, remote-ready signal
│   │   ├── Skills.tsx      ← 4-category skill grid
│   │   ├── Projects.tsx    ← Featured 4 projects (card grid)
│   │   ├── Experience.tsx  ← Timeline: SMIT, Saylani, Deloitte, HBL, ibex
│   │   ├── Certifications.tsx ← Badge layout, top certs only
│   │   └── Contact.tsx     ← Formspree form, social icons
│   └── ui/
│       ├── ThemeToggle.tsx ← Sun/Moon toggle, localStorage persistence
│       ├── ProjectCard.tsx ← Reusable card: title, tech, links
│       ├── SkillBadge.tsx  ← Pill badge for skill tags
│       ├── TimelineItem.tsx← Single experience row
│       └── SectionLabel.tsx← Eyebrow label (FEATURED PROJECTS, etc.)
│
├── data/
│   ├── projects.ts         ← All project data (title, desc, tech, links)
│   ├── skills.ts           ← Skills grouped by category
│   ├── experience.ts       ← Work/education timeline entries
│   └── certifications.ts  ← Cert name, issuer, date, badge URL
│
├── public/
│   ├── profile.jpg         ← Professional headshot (optimized, WebP preferred)
│   ├── resume.pdf          ← Downloadable CV (keep updated)
│   └── og-image.png        ← Open Graph image for social sharing (1200×630)
│
├── lib/
│   └── utils.ts            ← cn() helper (clsx + tailwind-merge)
│
├── tailwind.config.ts
├── tsconfig.json
├── next.config.ts
├── .env.local              ← FORMSPREE_ID (never commit this)
└── CLAUDE.md               ← This file
```

---

## Design System

### Color Tokens

All colors are defined as CSS variables in `globals.css`.
**Never hardcode hex values in components — always use token names.**

```css
/* Light mode — :root */
--color-bg:         #FAFAFA;    /* Page background           */
--color-surface:    #F3F4F8;    /* Cards, skill row bg        */
--color-accent:     #1740D4;    /* CTAs, links, badges        */
--color-accent-sub: #EEF0FF;    /* Accent background tint     */
--color-ink:        #080918;    /* Primary text               */
--color-muted:      #44445A;    /* Secondary text, subtitles  */
--color-border:     #E8E8EF;    /* Dividers, card borders     */
--color-card:       #FFFFFF;    /* Project card background    */

/* Dark mode — .dark */
--color-bg:         #0C0C16;
--color-surface:    #101018;
--color-accent:     #05D4B4;
--color-accent-sub: #0D2420;
--color-ink:        #F0F0FA;
--color-muted:      #8888AA;
--color-border:     #1E1E2E;
--color-card:       #101018;
```

### Tailwind Mapping

```ts
// tailwind.config.ts — extend.colors
colors: {
  bg:      'var(--color-bg)',
  surface: 'var(--color-surface)',
  accent:  'var(--color-accent)',
  'accent-sub': 'var(--color-accent-sub)',
  ink:     'var(--color-ink)',
  muted:   'var(--color-muted)',
  border:  'var(--color-border)',
  card:    'var(--color-card)',
}
```

### Typography

```ts
// tailwind.config.ts — extend.fontFamily
fontFamily: {
  heading: ['var(--font-heading)'],  // Lora (light) / JetBrains Mono (dark)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JamilRaza001/my-portfolio](https://github.com/JamilRaza001/my-portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
