---
trigger: always_on
description: Black Pride Directory is a Next.js 15 static site that serves as a directory for Black pride events across the US and globally. Events are stored as Markdown files with YAML frontmatter and optionally in a Strapi CMS backend. The two sources are merged at runtime and rendered via the App Router with full SSG support.
---

# Black Pride Directory — Claude Code Reference

## Project Overview

Black Pride Directory is a Next.js 15 static site that serves as a directory for Black pride events across the US and globally. Events are stored as Markdown files with YAML frontmatter and optionally in a Strapi CMS backend. The two sources are merged at runtime and rendered via the App Router with full SSG support.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Next.js 15.3.2 — App Router, Turbopack, SSG |
| Language | TypeScript 5 (strict mode) |
| Runtime | React 19 |
| Styling | Tailwind CSS v4 + PostCSS v4 |
| UI Components | shadcn/ui (New York style, Stone base) + Radix UI |
| Forms | React Hook Form + Zod |
| Animation | Framer Motion |
| Icons | Lucide React |
| Maps | Google Maps JS API (`@googlemaps/js-api-loader`) |
| Email | Resend API |
| CMS | Strapi v4.10+ (local dev only; no production instance yet) |
| Hosting | Vercel |

---

## Directory Structure

```
/
├── app/                        # Next.js App Router pages
│   ├── layout.tsx              # Root layout (nav, footer, analytics)
│   ├── page.tsx                # Homepage
│   ├── globals.css             # Global styles + Tailwind directives
│   ├── robots.ts               # robots.txt generation
│   ├── sitemap.ts              # sitemap.xml generation
│   ├── events/
│   │   ├── page.tsx            # Events listing — merges .md + Strapi sources
│   │   └── [slug]/page.tsx     # Event detail — SSG via generateStaticParams
│   ├── cities/
│   │   ├── page.tsx            # City listing
│   │   └── [slug]/page.tsx     # City detail
│   ├── blogs/
│   │   ├── page.tsx            # Blog listing — Strapi-only
│   │   └── [slug]/page.tsx     # Blog post detail
│   ├── post/page.tsx           # Event submission form (password-protected)
│   ├── about/page.tsx          # About page
│   └── api/post-event/route.ts # API route: forwards event submissions to Strapi
│
├── components/                 # React components
│   ├── ui/                     # shadcn/ui primitives (do not edit directly)
│   ├── events-grid-with-search.tsx  # Main events list: search, filter, view toggle
│   ├── event-card.tsx          # Individual event card
│   ├── event-form.tsx          # Event submission form (~700 lines)
│   ├── events-map.tsx          # Google Maps integration
│   ├── calendar-31.tsx         # Calendar month view
│   ├── add-to-calendar-button.tsx   # Google Calendar / iCal export
│   ├── blog-card.tsx           # Blog card
│   ├── blog-grid.tsx           # Blog grid with search + category filter
│   ├── navbar.tsx              # Top navigation
│   └── footer.tsx              # Site footer
│
├── lib/                        # Core utilities
│   ├── events.ts               # Markdown event loader (gray-matter)
│   ├── fetch.ts                # Strapi API client (fetchAll, fetchOne, postOne)
│   ├── collections.ts          # TypeScript types for all Strapi collections
│   ├── cities.ts               # Markdown city loader
│   ├── utils.ts                # Formatters, isStrapiImage guard, constants
│   ├── schemas.ts              # Zod validation schemas
│   └── actions.ts              # Next.js server actions
│
├── content/                    # Active Markdown content (READ BY APP)
│   ├── events/                 # Event .md files served by the app
│   └── cities/                 # City .md files (14 cities)
│
├── events_md/                  # STAGING — output of excel-converter.py
│                               # NOT read by the app; move files to content/events/
│
├── public/images/              # Static event and city images
│
├── excel-converter.py          # Excel → .md files (outputs to events_md/)
├── upload_to_strapi.py         # events_md/ → Strapi CMS via API
├── fetch_images.py             # Downloads og:image from event website URLs
├── patch_website_field.py      # Patches fields in existing .md files
├── blkoutlist.xlsx / .csv      # Source spreadsheet for bulk imports
│
├── next.config.ts              # Image remote patterns (11 external hosts)
├── tsconfig.json               # TypeScript config — path alias @/* → root
├── components.json             # shadcn/ui config
└── .env                        # Local secrets (never commit)
```

---

## Data Architecture

### Two Sources, One List

The events page (`app/events/page.tsx`) merges events from both sources:

```
content/events/*.md  ──── lib/events.ts (gray-matter)  ──┐
                                                           ├── merged array → EventsWithSearch
Strapi /api/events   ──── lib/fetch.ts (fetchAll)      ──┘
```

- Markdown events: slugs derived from filenames (without `.md`)
- Strapi events: `documentId` assigned as slug at merge time

Blogs are Strapi-only. Cities use both markdown (static data) and Strapi (not yet wired).

### Staging Pipeline for Bulk Imports

```
blkoutlist.xlsx
    → excel-converter.py   (generates events_md/*.md)
    → [manual review]
    → move to content/events/    ← for markdown serving

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jbowen4/black-pride-directory](https://github.com/jbowen4/black-pride-directory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
