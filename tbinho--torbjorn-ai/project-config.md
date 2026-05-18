---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **General rules:** See `C:\Dev\spitakolus\CLAUDE.md` for shared standards (documentation, assets, Meta Ads, infrastructure).

## Repository Identity

**This is torbjorn-ai** - Personal portfolio/documentation website for Torbjörn Sandblad.

- **Purpose:** Showcase AI-integrated work processes
- **Tech stack:** Next.js 15, TypeScript, Tailwind CSS, Keystatic
- **Production:** https://www.torbjornsai.site
- **Tech stack:** Next.js 15, TypeScript, Tailwind CSS, Keystatic CMS

**Related repos:**
- `spitakolus` → shared documentation (general rules)
- `flocken-website` → flocken.info product
- `nastahem` → nastahem.com product

---

## Commands

```bash
npm install          # Install dependencies
npm run dev          # Dev server (localhost:3000)
npm run build        # Production build
npm run lint         # ESLint
```

**Keystatic Editor:** Visit `localhost:3000/keystatic` to access the visual content editor.

---

## Project Structure

```
torbjorn-ai/
├── app/
│   ├── page.tsx              # Start / Om arbetet
│   ├── arbete/
│   │   ├── page.tsx          # Process list
│   │   └── [slug]/page.tsx   # Individual process
│   ├── riktning/page.tsx     # Riktning (static)
│   ├── keystatic/            # Keystatic admin UI
│   └── api/keystatic/        # Keystatic API routes
├── components/
│   ├── Header.tsx
│   └── Footer.tsx
├── content/
│   └── processer/            # MDX/Markdoc process files
├── public/assets/images/     # Images and placeholders
├── keystatic.config.ts       # Content schema
└── tailwind.config.js        # Design tokens
```

---

## Content Structure

### Process files (content/processer/)

Each process follows this exact structure:
1. Sammanfattning (in frontmatter)
2. Syfte & avgränsning
3. AI-first angreppssätt
4. Verktyg & system
5. Resultat / nuläge
6. Lärdomar
7. Fortsättning (with status)
8. Uppdateringar (timestamped)

### Ämneskluster (categories)

- `tracking-data-analys` – Tracking, data & analys
- `content-kreativ-produktion` – Content & kreativ produktion
- `automation-arbetsfloden` – Automation & arbetsflöden
- `beslutsstod-prioritering` – Beslutsstöd & prioritering
- `foretagsbyggande-ai` – Företagsbyggande med AI

### Status values

- `aktiv` – Currently being worked on
- `parkerad` – On hold, not abandoned
- `avslutad` – Completed

---

## Design System

### Colors (Tailwind)

```css
base: #FAFAF8         /* Warm off-white background */
text: #1A1A1A         /* Primary text */
text-muted: #6B7280   /* Secondary text */
accent: #3D6B5C       /* Primary accent (muted green) */
accent-hover: #4A7C6F /* Hover state */
border: #E5E7EB       /* Borders */
```

### Typography

- **Display/Headlines:** Georgia (serif) – personality, substance
- **Body:** Inter (sans-serif) – clean, readable
- **Code:** JetBrains Mono – technical elements

### Design Principles

- Text first, always
- Generous whitespace
- Portfolio feel, wiki simplicity
- Same template everywhere
- Professional, human, trustworthy

---

## Visuell stil och bildproduktion

Tre dokument styr bildarbetet:

- **`docs/visuell-stil.md`** — det estetiska språket: palett, halftone, referensbibliotek (Ref A, Ref B, Miljö 1-4 i `public/assets/references/`), filnamnskonventioner.
- **`docs/bildgenerering.md`** — produktionsflöde och Nano Banana-lärdomar: scen-prompt-mall, reference-usage-rules-block, brand-filter-strategier, edit vs fresh-generate, "exactly ONE"-mönstret.

Hero-bilder för artiklar sparas i `public/assets/images/` med filnamnet `{artikel-slug}-hero.jpg` — detekteras automatiskt av artikel- och listsidorna.

Referensbiblioteket i `public/assets/references/` används i alla framtida bildgenereringar. Bifoga alltid Ref A för karaktärslikhet + en miljöreferens för stil. Lägg till Ref B om scenen har en pratbubbla.

---

## Skrivguide

Den aktiva skrivguiden ligger alltid i `docs/skrivguide.md`. Det är ett levande dokument – ingen versionsarkivering, git-historiken är versionshanteringen.

**Uppdatera guiden:**
1. Redigera `docs/skrivguide.md` direkt
2. Commita med `git commit -m "Docs: update skrivguide – [kort beskrivning av ändringen]"`
3. Pusha till båda remotes

**Guiden täcker:** ton & röst, artikelstruktur, tooltip-syntax, NextThreshold-komponenten, formatregler och titelprincipen.

---

## Deployment

### Setup (hur det är konfigurerat)

Vercel Hobby-planen tillåter bara ett GitHub-konto per team. Torbjörns Vercel-team
är kopplat till **RaquelSandblad**-kontot. Därför deployas via ett privat spegelrepo:

- `origin` → `tbinho/torbjorn-ai` (backup/källkod)
- `raquel` → `RaquelSandblad/torbjorn-ai` (Vercel deployas härifrån)

Vercel är anslutet till `RaquelSandblad/torbjorn-ai` och deployas automatiskt vid push.

### Deploy-kommando

```bash
git add .
git commit -m "Description"
git push origin main   # backup
git push raquel main   # triggar Vercel-deploy
```

### Viktigt: commit-author MÅSTE vara RaquelSandblad

Vercel Hobby-planen tillåter inte collaborators på privata repos. Det betyder
att commits med en annan author (t.ex. `tbinho`) blockas av Vercel med:

> "Deployment blocked: The commit author did not have contributing access to

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tbinho/torbjorn-ai](https://github.com/tbinho/torbjorn-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
