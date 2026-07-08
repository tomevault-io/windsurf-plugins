---
trigger: always_on
description: This file is the single source of truth for any Claude Code agent working on this
---

# ThreatDex — Claude Code Agent Instructions

This file is the single source of truth for any Claude Code agent working on this
repository. Read it completely before touching any code. Every task handed to you
will reference a phase and issue number from the task list at the bottom.

-----

## 1. Project overview

ThreatDex is a web app that aggregates cyber threat actor intelligence from public
CTI feeds (MITRE ATT&CK, ETDA, AlienVault OTX, MISP, OpenCTI) and renders each
actor as an interactive "trading card" — flippable front/back with stats, TTPs,
campaigns, tools, and references.

**Tagline:** Know your adversaries, card by card.

-----

## 2. Project structure

```
threatdex/
├── app/                      # React Router v7 application
│   ├── components/           # Card UI components
│   ├── lib/
│   │   ├── supabase.client.ts  # Browser Supabase client
│   │   └── supabase.server.ts  # Server-side Supabase client
│   ├── routes/
│   │   ├── _index.tsx        # Home page — card grid + search/filters
│   │   └── actors.$id.tsx    # Actor detail page
│   ├── schema/
│   │   └── index.ts          # Zod schemas + TypeScript types (canonical data model)
│   ├── app.css               # Tailwind CSS v4 + brand design system
│   ├── root.tsx
│   └── routes.ts
├── workers/                  # TypeScript data ingestion scripts (run via tsx)
│   ├── mitre-sync.ts         # MITRE ATT&CK STIX bundle ingestion
│   ├── etda-sync.ts          # ETDA APT scraper
│   ├── otx-sync.ts           # AlienVault OTX connector
│   ├── image-gen.ts          # AI hero image generation
│   └── shared/               # Shared utilities
│       ├── supabase.ts       # Supabase admin client for workers
│       ├── models.ts         # Shared TypeScript data models
│       ├── dedup.ts          # Alias deduplication + actor merge logic
│       └── rarity.ts         # Rarity tier + threat level computation
├── supabase/
│   └── migrations/           # PostgreSQL schema + RLS policies
├── tests/
│   ├── components/           # Vitest component tests
│   ├── e2e/                  # Playwright smoke tests
│   └── workers/              # Worker unit tests
├── .github/
│   └── workflows/
│       ├── ci.yml            # Test + lint on push/PR
│       └── sync.yml          # Nightly data sync cron
├── docs/
│   ├── API.md
│   ├── DATA_SOURCES.md
│   └── ARCHITECTURE.md
├── .env.example
├── CLAUDE.md                 ← this file
├── CONTRIBUTING.md
├── SECURITY.md
├── netlify.toml              # Netlify deployment (edge SSR)
└── README.md
```

-----

## 3. Tech stack

| Layer           | Technology                                                      |
|-----------------|-----------------------------------------------------------------|
| Frontend        | React Router v7 (Vite SSR), TypeScript, Tailwind CSS v4        |
| Database        | PostgreSQL 15 via Supabase (managed, auto-REST)                 |
| Data access     | Supabase JS SDK (`@supabase/supabase-js`) in React Router loaders |
| Workers         | TypeScript scripts executed via `tsx` in GitHub Actions         |
| Package manager | pnpm (single `package.json`, no workspaces)                     |
| Testing         | Vitest (unit), Playwright (e2e)                                 |
| Linting         | ESLint + TypeScript-ESLint                                      |
| CI              | GitHub Actions                                                  |
| Hosting         | Netlify (edge SSR via `@netlify/vite-plugin-react-router`)      |

-----

## 4. Brand + design system

Apply these values consistently across all frontend work.

```
--color-wiz-blue:       #0254EC   /* primary CTAs, nav */
--color-purplish-pink:  #FFBFFF   /* accents, highlights */
--color-cloudy-white:   #FFFFFF   /* backgrounds, card surfaces */
--color-serious-blue:   #00123F   /* page background, card borders */
--color-blue-shadow:    #173AAA   /* secondary nav, card frames */
--color-sky-blue:       #6197FF   /* links, tags */
--color-light-sky-blue: #978BFF   /* subtle accents */
--color-pink-shadow:    #C64BA4   /* hover states */
--color-vibrant-pink:   #FF0BBE   /* rarity badges, "Dex" logotype */
--color-frosting-pink:  #FFBFD6   /* soft backgrounds */
--color-surprising-yellow: #FFFF00 /* MYTHIC tier glow, warnings */
```

Typography: monospace for all data/stat fields, a clean sans-serif for body copy.
Overall feel: dark navy base, vivid blue/pink accents — Wiz-style, optimistic security.

-----

## 5. Data model (canonical)

All ingestion workers and UI components must conform to this schema.
The authoritative TypeScript definition lives in `app/schema/index.ts`.

```typescript
interface ThreatActor {
  id: string                    // slug, e.g. "apt28"
  canonicalName: string
  aliases: string[]
  mitreId?: string              // e.g. "G0007"
  country?: string
  countryCode?: string          // ISO 3166-1 alpha-2
  motivation: Motivation[]      // "espionage" | "financial" | "sabotage" | "hacktivism" | "military"
  threatLevel: number           // 1–10
  sophistication: Sophistication
  firstSeen?: string            // YYYY
  lastSeen?: string             // YYYY
  sectors: string[]
  geographies: string[]
  tools: string[]
  ttps: TTPUsage[]
  campaigns: Campaign[]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adilio/threatdex](https://github.com/adilio/threatdex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
