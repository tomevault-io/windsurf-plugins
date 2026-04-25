---
trigger: always_on
description: Authoritative reference for Claude when working in this repository.
---

# GoalBet — CLAUDE.md

Authoritative reference for Claude when working in this repository.
Read this before touching any file. Everything here reflects the live codebase.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Tech Stack](#2-tech-stack)
3. [Commands](#3-commands)
4. [Critical Rules](#4-critical-rules)
5. [File & Folder Map](#5-file--folder-map)
6. [Routing](#6-routing)
7. [Architecture Overview](#7-architecture-overview)
8. [Auth System](#8-auth-system)
9. [Sync System](#9-sync-system)
10. [Scoring System](#10-scoring-system)
11. [Coin Economy](#11-coin-economy)
12. [Match Status System](#12-match-status-system)
13. [League System & ESPN Coverage](#13-league-system--espn-coverage)
14. [Database & Migrations](#14-database--migrations)
15. [Theme System](#15-theme-system)
16. [Internationalisation (i18n)](#16-internationalisation-i18n)
17. [Stores](#17-stores)
18. [Hooks](#18-hooks)
19. [CI / GitHub Actions](#19-ci--github-actions)
20. [Admin Console](#20-admin-console)
21. [Common Pitfalls](#21-common-pitfalls)

---

## 1. Project Overview

**GoalBet** is a non-commercial football prediction game for friend groups.
Users sign up with email/password or Google, join private groups via invite code, predict match outcomes across 5 tiers, stake coins, and compete on a real-time leaderboard.

**Live URL:** Auto-deploys from `main` via Vercel.
**Supabase project:** `rzavwyejcldvztkykhks`
**Backend:** Hosted on Render (free tier — sleeps after ~15 min inactivity).
**Primary market:** Israel (Hebrew-first, RTL support, midnight Israel timezone for daily bonuses).
**Zero paid APIs:** ESPN public scoreboard (no key), Supabase free tier, Render free tier.

---

## 2. Tech Stack

| Layer | Technology | Version |
|---|---|---|
| Build | Vite | ^5.0 |
| UI | React | ^18.2 |
| Language | TypeScript | ^5.3 (strict) |
| Styling | Tailwind CSS | ^3.4 |
| Animation | Framer Motion | ^12.36 |
| Routing | react-router-dom | ^6.21 |
| State | Zustand | ^4.5 |
| Backend/DB | Supabase JS | ^2.39 |
| Icons | Lucide React | ^0.577 |
| Backend runtime | Node.js + Express | ^4.18 |
| Backend cron | node-cron | ^3.0 |
| Backend HTTP client | axios | ^1.6 |

**No** Redux, MUI, Chakra, styled-components, class-based components, or TheSportsDB (replaced by ESPN).

---

## 3. Commands

```bash
# Frontend — http://localhost:5173
cd frontend && npm run dev

# Backend — http://localhost:3001
cd backend && npm run dev

# Type-check (run before committing)
cd frontend && npx tsc --noEmit

# Build (what CI runs — must pass)
cd frontend && npm run build

# Manual match sync (dev only)
cd backend && npm run sync

# Seed dev data
cd backend && npm run seed

# Deploy all pending migrations to remote Supabase
supabase db push --linked

# Check migration status (what's applied vs pending)
supabase migration list --linked
```

**Required env files** — never committed to git:

```
frontend/.env.local
  VITE_SUPABASE_URL=https://rzavwyejcldvztkykhks.supabase.co
  VITE_SUPABASE_ANON_KEY=<anon key>
  VITE_BACKEND_URL=http://localhost:3001

backend/.env
  SUPABASE_URL=https://rzavwyejcldvztkykhks.supabase.co
  SUPABASE_SERVICE_ROLE_KEY=<service role key>
  PORT=3001
  NODE_ENV=development
```

No feature flags required. Email + password auth is active by default; no env var needed.

### Auto-migration hook

`.claude/settings.local.json` contains a PostToolUse hook that automatically runs
`supabase db push --linked` whenever Claude writes or edits a file matching
`supabase/migrations/*.sql`. This requires the Supabase CLI to be authenticated:

```bash
supabase login   # one-time browser auth — do this before working on migrations
```

After login, every migration file Claude writes is auto-deployed to the remote DB.

---

## 4. Critical Rules

These have caused build failures or subtle bugs in the past. Never violate them.

### 4.1 Framer Motion `ease` type

```tsx
// ❌ Breaks tsc — number[] is not assignable to Easing in some contexts
transition={{ ease: [0.4, 0, 1, 1] }}

// ✅ Correct
transition={{ ease: 'easeIn' as const }}
transition={{ ease: 'easeOut' as const }}
transition={{ ease: 'easeInOut' as const }}
```

### 4.2 Framer Motion `Variants` — import the type

```tsx
import { type Variants } from 'framer-motion'

const slideVariants: Variants = {
  enter: (dir: number) => ({ x: dir > 0 ? 28 : -28, opacity: 0 }),
  center: { x: 0, opacity: 1 },
  exit:   (dir: number) => ({ x: dir > 0 ? -28 : 28, opacity: 0,
    transition: { duration: 0.14, ease: 'easeIn' as const } }),
}
```

### 4.3 AppShell owns ALL auto-sync — never add sync to page components

```tsx
// ❌ Creates race conditions and infinite spinners
useEffect(() => { fetch('/api/sync/matches') }, [])  // inside a page component

// ✅ AppShell.tsx is the only file that triggers automatic sync
// Pages dispatch 'goalbet:synced' listeners — never the triggerers
```

### 4.4 Realtime UPDATE events — always re-fetch, never swap in-place

```tsx
// ❌ The Realtime payload is PARTIAL — JSONB columns (predicted_*, add_ons) are missing
.on('postgres_changes', { event: 'UPDATE', ... }, (payload) => {
  setMatches(prev => prev.map(m => m.id === payload.new.id ? payload.new : m))
})

// ✅ Always call a full re-fetch

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Roychen651) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
