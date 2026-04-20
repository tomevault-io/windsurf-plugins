---
trigger: always_on
description: Naruto Daily is a daily character-guessing game inspired by Narutodle. Players guess a Naruto character
---

# Naruto Daily — Claude Code Context

## Project Overview

Naruto Daily is a daily character-guessing game inspired by Narutodle. Players guess a Naruto character
based on attribute feedback (classic Wordle-style). One puzzle per day, determined by a deterministic
date-based seed.

## Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Next.js (TypeScript, App Router) |
| Styling | Tailwind CSS + shadcn/ui |
| Deploy | Vercel |
| Data | `data/characters.json` — static, versioned in repo |
| Scraper | Python (BeautifulSoup + requests / MediaWiki API) |
| Language | English (i18n architecture mapped for future use) |

## Project Structure

```
naruto-daily/
├── CLAUDE.md
├── README.md
├── docs/
│   ├── ARCHITECTURE.md
│   ├── DATA-SCHEMA.md
│   ├── SCRAPER.md
│   ├── GAME-MECHANICS.md
│   └── ROADMAP.md
├── data/
│   ├── characters.json          # game-ready, from filter script — DO NOT edit manually
│   ├── characters-raw.json      # scraper output (gitignored)
│   └── canon-arcs.json          # manual whitelist of canonical arcs
├── scripts/
│   └── filter-characters.ts     # TypeScript filter: raw → game-ready
├── scraper/
│   ├── main.py
│   ├── requirements.txt
│   └── README.md
├── src/
│   ├── app/                     # Next.js App Router pages
│   ├── components/              # React components
│   ├── lib/
│   │   ├── daily.ts             # daily seed logic
│   │   ├── game.ts              # attribute comparison logic
│   │   └── storage.ts           # localStorage helpers
│   └── types/
│       └── character.ts         # TypeScript types
├── public/
├── package.json
├── tsconfig.json
└── next.config.ts
```

## Essential Commands

```bash
# Development
npm run dev          # Start Next.js dev server (http://localhost:3000)
npm run build        # Production build
npm run lint         # ESLint
npm run type-check   # TypeScript strict check (tsc --noEmit)

# Scraper — IMPORTANT restrictions for agents
# NEVER run the full scraper to validate implementations
# Only run for specific characters or with a limited character count:
npm run scrape           # Full scrape → data/characters-raw.json (slow, network)
npm run filter-data      # Filter raw → data/characters.json (fast, local)
npm run build-data       # Full pipeline: scrape + filter
npm run validate-data    # Validate characters.json schema

# Agent-safe scraping (NEVER run full scrape to validate implementations):
npm run scrape-test                                        # limit 5 characters
cd scraper && python main.py --limit 5                    # equivalent
cd scraper && python main.py --characters "Naruto Uzumaki,Sasuke Uchiha"
```

## UI & React Best Practices

When writing or reviewing React/Next.js UI code, invoke the `vercel-react-best-practices` skill before implementing. Key rules to follow:
- Prefer Server Components; use `'use client'` only when necessary
- Avoid hydration mismatches — never read `localStorage`/`window` in `useState` initializers; use `useEffect`
- Load client-only state after hydration to prevent SSR/client HTML mismatch
- Use `whitespace-normal` in nested components to override ancestor `whitespace-nowrap`

## Plans

Whenever the user asks to create a plan, create a `.md` file for it in `docs/plans/` following the naming convention `YYYY-MM-DD-<feature-name>.md`. This is the default path for all implementation plans.

When executing a plan, follow every step exactly as written — including the git workflow (worktree creation, branch naming, commit order, PR creation). Do not skip or reorder steps. If a step cannot be completed, stop and explain why rather than silently skipping it.

**IMPORTANT for plan agents**: At the top of every plan file, include this instruction for the executing agent:
> Before touching any files, read `CLAUDE.md` in the project root to understand conventions, constraints, and workflow rules.

## Conventions

### TypeScript
- Strict mode enabled (`tsconfig.json`: `"strict": true`)
- All types live in `src/types/`
- No `any` — use proper types or `unknown`
- Use `interface` for data shapes, `type` for unions/aliases

### Components
- All React components in `src/components/`
- Use shadcn/ui base components (Button, Input, Badge, Dialog, etc.)
- Prefer Server Components; use `'use client'` only when necessary
- Component files: PascalCase (e.g., `GuessRow.tsx`)
- Avoid nested conditionals in render logic — extract each case into a named sub-component or helper function with early returns

### Data
- **NEVER modify `data/characters.json` manually** — regenerate via scraper
- **NEVER add filler-only characters** — only characters from canonical manga arcs
- Canonical arc whitelist: `data/canon-arcs.json`
- **Franchise scope**: only Naruto Part I and Naruto Shippuden (manga ch 1-700). Boruto, movie-only, and filler-only characters are excluded. The `series` field in `canon-arcs.json` enforces this at scraper load time.
- All string values in English, lowercase-normalized where applicable
- Array fields (e.g., `naturTypes`, `jutsuTypes`) must be sorted alphabetically

### Styling
- Tailwind utility classes only — no custom CSS files unless unavoidable
- Dark mode supported via Tailwind `dark:` prefix

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ArthurEnrique15) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
