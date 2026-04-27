---
trigger: always_on
description: When the user says 'autopilot', 'auto-pilot', or 'yolo mode': proceed with ALL tasks without asking for confirmation. Do NOT enter plan mode — execute directly. Do NOT use Task sub-agents unless explicitly asked. Apply edits, run commands, and keep moving until the goal is complete or you hit a genuine blocker.
---

# Aanvikshiki Website — Claude Context

## Autopilot Mode
When the user says 'autopilot', 'auto-pilot', or 'yolo mode': proceed with ALL tasks without asking for confirmation. Do NOT enter plan mode — execute directly. Do NOT use Task sub-agents unless explicitly asked. Apply edits, run commands, and keep moving until the goal is complete or you hit a genuine blocker.

## CHANGELOG Rule
- **Single CHANGELOG**: `CHANGELOG.md` at the workspace root is the ONLY changelog file. Do NOT create or update changelogs in sub-project folders.
- **Always update before committing**: After implementing any code changes, update `CHANGELOG.md` under `## [Unreleased]` BEFORE committing. Include dated section headers, what was added/changed/fixed, key files and endpoints.
- **Never auto-push**: After committing, ask the user before running `git push`.

## Claude Operational Guidelines

1. **NEVER report incomplete tasks as complete** — Always verify task completion before marking done
2. **DO NOT worry about context window limits** — If needed, ask user to continue in a new session to complete the task
3. **ALWAYS keep responses brief but clear** — Concise communication, no unnecessary verbosity
4. **NEVER ask user to approve anything in auto-pilot mode** — Maintain and keep updating allowed command list in `.claude/settings.local.json`
5. **Maintain full context awareness** across component and page boundaries
6. **No unsolicited status documents** — Show status in conversation only
7. **Factual correctness is paramount** — Be factual with evidence, never infer without stating
8. **Requirement deviation alert** — ALERT user and update docs BEFORE implementing deviations from requirements
9. **Validate before declaring done** — Run `npm run build` (next build + next-sitemap) before marking any task complete. Do not report success without verification.
10. **Clarify ambiguity** — When requirements are ambiguous, ask clarifying questions rather than guessing intent.

## Project Overview
**Aanvikshiki Website** — A corporate/consulting website built with Next.js (App Router, static export), React, and Tailwind CSS. Features include a home page, services page, work/case studies, approach methodology, about page, contact form, and an "Ease" product page. Uses Next.js file-system routing with static generation, motion (Framer Motion) for animations, shadcn/ui + Radix UI for component primitives, and Sanity CMS for content management. Deployed to Cloudflare Pages.

## Primary Stack
- **Framework:** Next.js 15 (App Router, `output: 'export'`)
- **Language:** TypeScript 5.x (strict mode)
- **UI Library:** React 18
- **Styling:** Tailwind CSS 4 + shadcn/ui + Radix UI primitives
- **Animations:** Motion (Framer Motion)
- **Routing:** Next.js App Router (file-system based)
- **CMS:** Sanity (headless)
- **Icons:** Lucide React
- **SEO:** Next.js Metadata API + next-sitemap
- **Hosting:** Cloudflare Pages (static export)
- **Package Manager:** npm

Always assume this stack unless told otherwise.

## Tech Stack
| Component | Technology |
|-----------|------------|
| Language | TypeScript 5.x |
| Framework | Next.js 15 (App Router, static export) |
| UI Library | React 18 |
| Styling | Tailwind CSS 4, shadcn/ui, Radix UI |
| Animations | Motion (Framer Motion) |
| Routing | Next.js App Router |
| CMS | Sanity (headless) |
| Icons | Lucide React |
| Carousel | Embla Carousel |
| Hosting | Cloudflare Pages |

## Architecture

```
aanvikshikiWebsite/
├── src/
│   ├── index.css                # Global styles + Tailwind imports
│   ├── app/                     # Next.js App Router
│   │   ├── layout.tsx           # Root layout (fonts, metadata, Navbar, Footer, ThemeProvider)
│   │   ├── page.tsx             # Home page route
│   │   ├── not-found.tsx        # 404 page
│   │   ├── services/page.tsx    # Services route
│   │   ├── approach/page.tsx    # Approach route
│   │   ├── ease/page.tsx        # Ease route
│   │   ├── contact/page.tsx     # Contact route
│   │   ├── about/page.tsx       # About route
│   │   ├── work/page.tsx        # Work listing route
│   │   └── work/[slug]/page.tsx # Dynamic case study route (generateStaticParams)
│   ├── views/                   # Client-side page components
│   │   ├── Home.tsx             # Landing page
│   │   ├── Services.tsx         # Services listing
│   │   ├── Work.tsx             # Our work / portfolio
│   │   ├── CaseStudy.tsx        # Case study detail (props-based)
│   │   ├── Approach.tsx         # Methodology / approach page
│   │   ├── About.tsx            # About the company
│   │   ├── Contact.tsx          # Contact form
│   │   ├── Ease.tsx             # Ease product page
│   │   └── NotFound.tsx         # 404 page
│   ├── components/
│   │   ├── Navbar.tsx           # Site navigation with dropdown mega menu
│   │   ├── Footer.tsx           # Site footer
│   │   ├── theme-provider.tsx   # next-themes wrapper
│   │   ├── hooks/               # Custom React hooks
│   │   ├── sections/            # Reusable page sections

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aaspl-omni) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
