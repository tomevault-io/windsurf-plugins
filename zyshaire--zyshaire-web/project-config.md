---
trigger: always_on
description: Zyshaire is a SaaS marketing/agency website built with React + TypeScript, deployed on Firebase via GitHub Actions. The site showcases AI consulting and web development services.
---

# CLAUDE.md — Zyshaire Project Knowledge Base

## Project Overview

Zyshaire is a SaaS marketing/agency website built with React + TypeScript, deployed on Firebase via GitHub Actions. The site showcases AI consulting and web development services.

---

## Repository Structure

```
zyshaire/
├── .github/workflows/             # CI/CD (Firebase deploy on merge + PR preview)
├── ui-designs/                    # Reference HTML mockups (9 static pages)
├── website/                       # React + TypeScript app (all development happens here)
└── zyshaire_project_knowledge.md  # Additional project context
```

All source code lives under `website/`. Run all npm commands from `website/`.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | React 19.2.0 |
| Language | TypeScript ~5.9.3 (strict) |
| Build | Vite 7.2.4 |
| Routing | React Router DOM 7.13.0 |
| Styling | Tailwind CSS 4.1.18 (via `@tailwindcss/vite`) |
| Forms | Formspree (`@formspree/react` 3.0.0) |
| Hosting | Firebase Hosting (`zyshaire-web` project) |
| CI/CD | GitHub Actions |

---

## Key File Map

### Configuration
| File | Purpose |
|------|---------|
| `website/package.json` | Dependencies, npm scripts (`dev`, `build`, `lint`, `preview`) |
| `website/vite.config.ts` | Vite with React + Tailwind plugins |
| `website/index.css` | Tailwind import + all design tokens (`@theme {}`) |
| `website/.env` | Formspree form IDs (not committed — see CI workflow for keys) |
| `website/firebase.json` | Firebase hosting: `dist/` dir, SPA rewrite all routes → `/index.html` |
| `website/.firebaserc` | Firebase project ID: `zyshaire-web` |
| `website/tsconfig.app.json` | TypeScript: ES2022, DOM, strict, `noUnusedLocals`, `noUnusedParameters` |

### Core Application
| File | Purpose |
|------|---------|
| `website/src/main.tsx` | App entry: StrictMode → Formspree Provider → BrowserRouter → App |
| `website/src/App.tsx` | All routes inside `<MainLayout>` |
| `website/src/types.ts` | TypeScript interfaces: `NavItem`, `StatItem`, `ServiceItem`, `CaseStudyItem`, `FaqItem` |
| `website/src/data/site-data.ts` | **All site content**: nav, stats, services, case studies, FAQs |

### Layout Components (`website/src/components/layout/`)
| File | Purpose |
|------|---------|
| `MainLayout.tsx` | Wraps `<Header>` + `<main>` + `<Footer>` |
| `Header.tsx` | Sticky nav, logo, mobile toggle, "Book a Call" CTA |
| `Footer.tsx` | 4-column footer: Services, Company, Legal links + copyright |

### Common Components (`website/src/components/common/`)
| File | Purpose |
|------|---------|
| `Container.tsx` | `max-w-7xl` responsive wrapper |
| `PageHero.tsx` | Hero: badge, title, highlight text, description, dual CTAs, grid background |
| `SectionHeading.tsx` | Eyebrow label + section title + description |
| `ServiceCard.tsx` | Service: title, description, problems[] list, deliverables[] list |
| `CaseStudyCard.tsx` | Case study: industry, challenge, solution, metrics |
| `StatsStrip.tsx` | Statistic value + label display |
| `FaqAccordion.tsx` | Collapsible FAQ using HTML `<details>` element |
| `CtaBanner.tsx` | Dark CTA section with grid pattern background |

### Utilities (`website/src/lib/`)
| File | Purpose |
|------|---------|
| `cn.ts` | Class name utility (filter falsy + join — simple, no `clsx` dependency) |
| `formspree.ts` | `getFormspreeFormId()`, `isFormspreeConfigured()` — reads from env vars |

---

## Routing

Defined in `website/src/App.tsx`. All routes wrapped in `<MainLayout>`.

| Route | Page File | Notes |
|-------|-----------|-------|
| `/` | `pages/HomePage.tsx` | Hero, stats, services, process, why Zyshaire, CTA |
| `/services` | `pages/ServicesPage.tsx` | Service details, engagement models, FAQs |
| `/ai-audit` | `pages/AIAuditPage.tsx` | Audit deliverables, 5-step process, audit inquiry form |
| `/about` | `pages/AboutPage.tsx` | Mission, principles, delivery model |
| `/insights` | `pages/InsightsPage.tsx` | Topic filter, 6 articles, newsletter/playbook form |
| `/contact` | `pages/ContactPage.tsx` | Contact options sidebar, 7-field project inquiry form |
| `/legal` | Redirects → `/legal/privacy` | |
| `/legal/:docType` | `pages/LegalPage.tsx` | `privacy`, `terms`, `cookies` docs with sidebar nav |
| `*` | `pages/NotFoundPage.tsx` | 404 page |

**Disabled route:** Case Studies page (`pages/CaseStudiesPage.tsx`) — file exists but route is commented out in `App.tsx`. Also removed from nav, footer, homepage, and insights.

---

## Design System

All tokens defined in `website/index.css` under `@theme {}`:

| Token | Value |
|-------|-------|
| Primary color | `#137fec` (blue) |
| Background light | `#f6f7f8` |
| Text main | `#0d141b` |
| Text muted | `#4c739a` |
| Font display | Space Grotesk |
| Font body | Manrope |
| Shadow glow | `0 16px 36px -14px rgba(19,127,236,0.45)` |

Use Tailwind utility classes exclusively (no inline styles). Responsive breakpoints: `md:`, `lg:`, `xl:`.

---

## Forms & Formspree Integration

Three forms in the app, all using `useForm` hook from `@formspree/react`:

| Form | Page | Env Var | Form ID |
|------|------|---------|---------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zyshaire/zyshaire-web](https://github.com/zyshaire/zyshaire-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
