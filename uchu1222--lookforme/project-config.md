---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LookForMe (LFM-DOM) is a NYC-focused mobile web app for location-based missed connections. Users post about meaningful encounters at specific places (restaurants, parks, transit) and others can find, match, and connect privately via a verification gate. Think Hinge meets a map, not Craigslist missed connections.

There are two implementations in this repo:
1. **`app/`** — The modern Vite + React + TypeScript app (active development)
2. **`old/LFMJesse.html`** — The original single-file prototype (~4,500 lines, `React.createElement` without JSX)

## Commands

### Modern app (`app/`)

```bash
cd app
npm install          # Install dependencies
npm run dev          # Dev server on http://localhost:5173
npm run build        # TypeScript check + production build (tsc -b && vite build)
npm run lint         # ESLint
npm run test         # Run tests (vitest)
npm run preview      # Preview production build
```

Requires `VITE_MAPBOX_TOKEN`, `VITE_SUPABASE_URL`, and `VITE_SUPABASE_ANON_KEY` in `app/.env` (see `app/.env.example`). Search also requires `OPENAI_API_KEY` set as a Supabase secret (not a Vite env var).

### Prototype (legacy, `old/`)

```bash
open old/LFMJesse.html                # Open directly in browser
cd old && python3 -m http.server 8000 # Serve locally (needed for Google Maps)
```

No build step. Loads React 18 + Tailwind from CDN. Google Maps API key is embedded in the HTML.

## Architecture (Modern App)

**Stack**: Vite 7, React 19, TypeScript 5.9, Tailwind 4, Mapbox GL 3.18, Zustand 5, Framer Motion 12, React Router 7, Supabase (Postgres + PostGIS + Auth + Realtime).

### Routing (`App.tsx`)

```
/                   → redirects to /discover
/discover           → DiscoverPage (map + pin card popover on desktop, bottom sheet feed on mobile)
/post               → PostPage (creation method: Pin / Establishment / Transit)
/post/pin           → PostPinPage (pin-on-map + post form)
/post/establishment → PostEstablishmentPage (place search + post form)
/messages           → MessagesPage (conversation list)
/messages/:id       → ChatPage (threaded chat)
/profile            → ProfilePage (placeholder)
```

### State (Zustand stores in `src/stores/`)

Six stores keep state separate from components (designed to be React Native portable):

- **`auth.ts`** — Supabase Auth (OTP email login), `user`, `session`, `profile`, `pendingAction`, `isAuthSheetOpen`. Uses immer middleware. Initializes on app mount via `App.tsx`.
- **`navigation.ts`** — `activeTab`, `createPostMode`
- **`map.ts`** — `selectedDate` (nullable, null = all dates), `genderFilter`, `selectedPost`, `activePostIndex`, `showFeedPanel`, `transitMode`, `viewTransit`
- **`postCreation.ts`** — `droppedPin`, `selectedPlace`, `content`, `seeking`, verification Q&A
- **`conversations.ts`** — Message threads and conversation state
- **`search.ts`** — `query`, `results` (null = inactive, [] = no matches), `isSearching`, `error`

### Layout

`AppShell` wraps all routes in a flex container with `TabBar` always rendered at the bottom. The tab bar reads and writes `activeTab` from the navigation store and syncs with React Router location.

### Discover Page (Desktop vs Mobile)

**Desktop (>=1024px):** Map-first layout — the map fills the full viewport with no sidebar. Clicking a pin opens a `MapPostCard` popover fixed at bottom-center of the map (Apple Maps style). Arrow keys cycle through posts (wrapping), map pans to follow. Escape or click-outside dismisses. A floating `FilterDropdown` sits at top-right of the map. Active pin scales up with a drop shadow. The `DesktopTopBar` has a live `SearchBar` component. Components: `MapPostCard` (card popover), `FilterDropdown` (floating filter), `SearchBar` (unified search input).

**Mobile (<1024px):** `PostRail` slides up as a bottom sheet (`max-h-[60vh]`, rounded top) toggled by a `FeedToggleButton` on the map. Contains a scrollable card list (`FeedPostCard` items) with sort toggle. Selected post detail + "Was This You?" button appears at bottom of the sheet. `FilterDropdown` lives in the mobile brand header. A search icon in the header expands to a `SearchBar` via Framer Motion slide-in; Cancel collapses back to the wordmark.

The `useIsDesktop` hook (`src/hooks/useMediaQuery.ts`, 1024px breakpoint) gates which experience renders. Both read from the same Zustand `map` store, so resizing across the breakpoint is seamless.

### Data

Frontend data is currently hardcoded in `src/lib/constants.ts` — 145 sample posts across dates and transit modes, 2 sample conversations, subway line/color mappings, MTA colors, borough lists, rail lines. Types live in `src/lib/types.ts`. The same 145 posts are seeded into Supabase with pgvector embeddings (see `supabase/seed.sql`). Deterministic UUIDs map between client numeric IDs and Supabase UUIDs: post ID `N` → UUID `10000000-0000-0000-0000-{N zero-padded to 12}`.

### Backend (Supabase)

**Project ref**: `btjcqshfvnzmlcylhzsd` — hosted on Supabase (US East).

**Database schema** (6 tables, all with RLS):
- `users` — profiles, seeking preference, rate limiting, subscription tier

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Uchu1222) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
