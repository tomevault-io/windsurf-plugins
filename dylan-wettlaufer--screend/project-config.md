---
trigger: always_on
description: We are building Screend (screend.dev) — an AI-powered resume analyzer and optimizer targeting tech job seekers (software engineers, backend, frontend, devops, ML engineers). The full PRD is at `docs/PRD.md` — reference it whenever making product or architecture decisions.
---

# Screend — Cursor Rules

## Project context
We are building Screend (screend.dev) — an AI-powered resume analyzer and optimizer targeting tech job seekers (software engineers, backend, frontend, devops, ML engineers). The full PRD is at `docs/PRD.md` — reference it whenever making product or architecture decisions.

## Tech stack
- Framework: Next.js 14 App Router (not Pages Router — never use pages/)
- Language: TypeScript — strict mode, always type everything, never use `any`
- Styling: Tailwind CSS with custom design tokens (see below)
- Components: shadcn/ui as base components + Magic UI for animations
- Auth: Clerk — all auth logic goes through Clerk, never roll custom auth
- Database: Supabase (PostgreSQL) — service role key server-side only, never expose to client
- AI: Anthropic SDK — model is always `claude-sonnet-4-20250514`, temperature 0 for structured outputs
- Payments: Stripe — subscriptions only, webhooks sync to Clerk metadata
- Deployment: Vercel

## Folder structure
```
app/                    ← Next.js App Router pages and API routes
  (marketing)/          ← public marketing pages (landing, pricing)
  (app)/                ← protected app pages (dashboard, scan, history)
  api/                  ← API route handlers
    analyze/route.ts
    rewrite/route.ts
    webhooks/
      stripe/route.ts
      clerk/route.ts
    billing/
      create-checkout/route.ts
      portal/route.ts
components/
  ui/                   ← shadcn generated components (do not edit manually)
  scan/                 ← scan-specific components (ScoreRing, FeedbackItem, etc.)
  layout/               ← Navbar, Footer, Sidebar
  landing/              ← landing page sections
lib/
  supabase.ts           ← Supabase client setup
  anthropic.ts          ← Anthropic SDK setup
  parsers.ts            ← pdf-parse and mammoth resume parsing logic
  utils.ts              ← shared helper functions
  types.ts              ← all shared TypeScript types and interfaces
docs/
  PRD.md                ← full product requirements document
```

## Design system

### Color tokens
These are defined as CSS variables in `globals.css` and as Tailwind custom colors in `tailwind.config.ts`. Always use these — never hardcode hex values in components.
```
These are defined as CSS variables in `globals.css` and as Tailwind custom colors
via @theme inline. Always use these — never hardcode hex values in components.

--bg-base:        #0a0a0b   page background (zinc, near black)
--bg-surface:     #111113   cards, sidebars, panels
--bg-raised:      #18181b   elevated elements, inputs
--bg-hover:       #212126   hover states
--border:         #27272a   default borders (CSS var: --border-screend)
--border-strong:  #3f3f46   emphasized borders
--accent:         #3b82f6   primary accent — electric blue (CSS var: --accent-screend)
--accent-subtle:  #17171a   accent background — no blue tints, neutral only
--cyan:           #67e8f9   secondary accent — positive states, matched keywords
--text-primary:   #fafafa   primary text (pure white)
--text-secondary: #a1a1aa   muted text, labels
--text-tertiary:  #71717a   hints, placeholders, metadata
--danger:         #f87171   errors, high severity, missing keywords
--warning:        #fb923c   mid scores, medium severity
--success:        #67e8f9   good scores, matched keywords (same as cyan)

### Color usage rules — follow these strictly
- Blue (#3b82f6): buttons, score ring, progress bars, active states, links, focus rings
- Cyan (#67e8f9): matched keywords, accepted feedback items, hero headline accent line, positive indicators, checkmarks, success states
- Red (#f87171): missing keywords, high severity dots, error states, destructive actions
- Never use blue as a background fill — use --accent-subtle (#17171a) only
- Never hardcode any hex value in a component — always reference the CSS variable
- Primary button text is #ffffff not black
- Score bars use a single blue (#3b82f6) at varying opacity based on score strength:
  100% = perfect, 85% = good, 65% = mid, 40% = low, never below 30%
- Matched keyword pills: bg rgba(103,232,249,0.08), text #67e8f9, border rgba(103,232,249,0.2)
- Missing keyword pills: bg var(--bg-raised), text #f87171, border var(--border-strong)
- Accepted feedback items: bg rgba(103,232,249,0.04), border rgba(103,232,249,0.15)
```

### Typography
- Sans font: system-ui / Tailwind default sans stack — never use Inter, Roboto, or Arial
- Mono font: `font-mono` (Courier New) for score labels, keyword pills, metadata, stat readouts, any data-y text
- Font weights: 400 for body, 500 for headings and labels — never use 600 or 700
- Always sentence case — never ALL CAPS or Title Case in any UI copy
- No mid-sentence bolding in UI text

### Spacing and radius
- Card border radius: 12px (`rounded-card`)
- Element border radius: 8px (`rounded-element`)
- Pill border radius: 9999px (`rounded-pill`)
- Borders are always `0.5px` — never `1px` or thicker except focus rings
- No box shadows anywhere — depth comes from background color layering only
- No gradients anywhere in the UI


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dylan-wettlaufer/screend](https://github.com/dylan-wettlaufer/screend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
