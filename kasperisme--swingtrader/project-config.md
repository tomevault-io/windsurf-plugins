---
trigger: always_on
description: **News Impact Screener** — swing trading research platform connecting headlines to stocks for retail investors.
---

# swingtrader — Claude Code Project Guide

## Project Overview

**News Impact Screener** — swing trading research platform connecting headlines to stocks for retail investors.

Stack: Next.js (App Router) + TypeScript + Supabase (auth/DB) + Sanity (CMS) + Tailwind, deployed on Vercel.

Key directories:
- `code/ui/` — Next.js app
- `code/ui/sanity/` — Sanity studio + schemas
- `code/ui/lib/sanity/` — GROQ queries, types, Sanity client
- `code/ui/app/blog/` — Blog pages
- `code/ui/app/docs/` — Documentation pages
- `code/ui/components/` — Shared UI components

## Content Writing — Blog Posts & Documentation

### Always write two versions

Every blog post (`post`) and documentation page (`docPage`) in Sanity has two body fields:

| Field | Purpose |
|-------|---------|
| `body` | Full prose — complete sentences, SEO-friendly, conversational |
| `cavemanBody` | Compressed caveman version — same structure, ~70% fewer words |

**Whenever writing or editing blog or doc content, always produce both `body` and `cavemanBody`.**

### Use the caveman skill for `cavemanBody`

Invoke `.claude/skills/caveman/SKILL.md` when producing `cavemanBody` content.

Core caveman rules (quick ref):
- Drop: articles (a/an/the), filler (just/basically/really), hedging, pleasantries
- Keep: technical terms exact, code unchanged, numbers/data
- Pattern: `[thing] [action] [reason]. [next step].`
- Fragments OK. Short synonyms. Active voice only.

Example pair:

> **Normal:** "The news impact score is calculated by analyzing sentiment, volume, and asset relevance across multiple sources."
>
> **Caveman:** "News impact score = sentiment + volume + asset relevance. Multi-source."

### Sanity schema reference

```typescript
// postType.ts + docPageType.ts both have:
body:        blockContent   // full prose
cavemanBody: blockContent   // caveman-compressed prose (optional but always fill it)
```

GROQ queries already fetch both fields. The UI switches between them via `CavemanContent` component reading `CavemanModeProvider` context.

## Caveman Mode — UI

The caveman/businessman toggle is global (localStorage-backed via `lib/caveman-mode.tsx`). It appears in:
- Desktop header (`components/site-header.tsx`)
- Mobile nav drawer (`components/site-header-mobile-nav.tsx`)
- Docs sidebar (`app/docs/_components/docs-sidebar.tsx`)

`CavemanContent` (`components/caveman-content.tsx`) renders the appropriate body based on the context.

## Skills Available

| Skill | Use when |
|-------|---------|
| `caveman` | Writing `cavemanBody` for any blog post or doc page |
| `ui-ux-pro-max` | Designing or reviewing UI components, layouts, styles |
| `taste-skill` | Building any UI — enforces premium design standards, kills generic AI patterns |

## Sanity Studio

Mounted at `/studio`. Use Vision tool for GROQ queries.

Content types:
- `post` — Blog posts
- `docPage` — Documentation pages (grouped by `section`, ordered by `order`)
- `author`, `category` — Supporting types

---
> Source: [kasperisme/swingtrader](https://github.com/kasperisme/swingtrader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
