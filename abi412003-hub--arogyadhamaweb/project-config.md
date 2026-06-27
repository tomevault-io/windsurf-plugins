---
trigger: always_on
description: > Context handoff for stateless Claude Code sessions. Read this fully before touching anything.
---

# CLAUDE.md

> Context handoff for stateless Claude Code sessions. Read this fully before touching anything.

---

## What this is

Public website for **Arogyadhama** — a 350-bed integrative medicine hospital at S-VYASA Yoga University, Prashanti Kutiram, Bengaluru. Founded in the lineage of Padma Shri Dr. H. R. Nagendra. Forty years of practice, 30,000+ patients treated, 400+ indexed research papers.

This is the **public-facing marketing & enquiry site** — not the patient app, not the clinical app. It exists to:

1. Position Arogyadhama for donors, international patients, S-VYASA stakeholders
2. Capture OPD/IPD enquiries (form posts to ERPNext, eventually)
3. Showcase the five healing systems, eleven clinical departments, 177-therapy catalog
4. Drive traffic to the existing arogyadhama.com — eventually replacing it

**Not in scope here:** patient app (separate Next.js project — 58-screen blueprint), clinical app (already deployed at `arogyadhama-connect.vercel.app`), HIMS, ERPNext customization.

---

## Stack

```
Next.js 14.2.35 (App Router)
TypeScript 5.6.3
Tailwind 3.4.15 (custom design tokens)
React Three Fiber 8.17 + drei 9.114 + three 0.170
Framer Motion 11.11
GSAP 3.12 (loaded but not yet used heavily)
Lucide React (icons)
Node 24.x on Vercel

Hosted: Vercel (e-digivault team)
Repo: github.com/abi412003-hub/arogyadhamaweb
```

No Supabase. No mock data layer. No App Router server components doing fetches yet (intentional — the data is inline until ERPNext wiring lands).

---

## Live URLs

| Surface | URL |
|---|---|
| Production (team-aliased) | https://arogyadhamaweb-e-digivault.vercel.app |
| Latest deployment | https://arogyadhamaweb-kp9r3wp66-e-digivault.vercel.app |
| `arogyadhamaweb.vercel.app` | **Owned by another Vercel team — DO NOT rely on this URL** |
| Inspector | https://vercel.com/e-digivault/arogyadhamaweb |
| GitHub | https://github.com/abi412003-hub/arogyadhamaweb |

**Vercel Deployment Protection** is currently ON — only e-digivault team members can view. To make public: Settings → Deployment Protection → set to Disabled.

---

## Vercel & GitHub identifiers

```
Team:        e-digivault            team_H3smw5RN6X19PkPl7KE2p0C3
Project:     arogyadhamaweb         prj_PdHrBsFqLH8YyL4QFZ5AdhLzrVaW
GitHub repo: abi412003-hub/arogyadhamaweb
Default branch: main
```

**Commits MUST be authored as `Abinash <abi412003@gmail.com>`.** Vercel only auto-deploys on commits matching this email. Set git config when starting a session:

```bash
git config user.name "Abinash"
git config user.email "abi412003@gmail.com"
```

---

## Environment variables

Set in Vercel dashboard → Project Settings → Environment Variables. **Never commit actual values.** See `.env.example` for the full list:

```
ERPNEXT_API_TOKEN          # rotate from arogyadhama.m.frappe.cloud → API Access
NEXT_PUBLIC_ERPNEXT_URL    # https://arogyadhama.m.frappe.cloud
```

---

## Design system — "Sacred Modernism"

Editorial luxury (Aman / Six Senses tier) with sacred geometry undertones. Slow breathing motion, never kinetic.

**Type**
- Display: Cormorant Garamond (with italic for emphasis)
- Body: DM Sans
- Sanskrit: Tiro Devanagari Hindi
- Mono / data: JetBrains Mono

All loaded via `next/font/google` in `src/app/layout.tsx` and exposed as CSS variables.

**Palette (CSS variables in `globals.css`, Tailwind tokens in `tailwind.config.ts`)**

| Token | Hex | Usage |
|---|---|---|
| `forest-deepest` | `#0F2419` | Footer, deepest backgrounds |
| `forest-deep` | `#1A3A2E` | Dark sections |
| `forest` | `#2D5A3D` | Primary brand, CTAs, Yoga system |
| `forest-soft` | `#5C8A6E` | Naturopathy system |
| `gold` | `#C9A961` | Sacred accent, hairlines, Ayurveda system |
| `gold-pale` | `#E8D5A3` | Italic display emphasis |
| `cream` | `#F5EFE0` | Section canvas |
| `paper` | `#FAF7F0` | Body background |
| `ink` | `#1C1C1A` | Body text, Physiotherapy system |
| `terracotta` | `#B8694A` | Acupuncture system, sparing accent |

**Motion vocabulary**
- Easing: `cubic-bezier(0.16, 1, 0.3, 1)` (out-expo) for entrances
- Durations: 700ms–1.6s for entrances, 200–400ms for hover
- Animations: `breathe` (6s loop), `drift` (24s), `rotate-slow` (60s), `fade-up`
- `prefers-reduced-motion: reduce` is honored globally

**Don't:** add new fonts. Don't use generic SaaS purples/blues. Don't add box-shadows that aren't `--shadow-soft` or `--shadow-deep`.

---

## File map

```
src/
├── app/
│   ├── layout.tsx              # Font loading, root nav + footer
│   ├── page.tsx                # Home composition (8 sections)
│   ├── globals.css             # CSS vars, paper-grain bg, gold-rule, eyebrow
│   ├── systems/page.tsx        # 5 systems + mandala + per-system editorial
│   ├── departments/page.tsx    # 11 clinical departments
│   ├── therapies/page.tsx      # Filterable catalog (Suspense + URL ?section=)
│   ├── campus/page.tsx         # Flythrough + campus details
│   ├── about/page.tsx          # S-VYASA history, timeline, leadership
│   └── contact/page.tsx        # Form + phone tree (currently placeholder POST)
│
├── components/
│   ├── ui/
│   │   ├── Navigation.tsx      # Editorial top bar, scroll bg-change, mobile drawer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [abi412003-hub/arogyadhamaweb](https://github.com/abi412003-hub/arogyadhamaweb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
