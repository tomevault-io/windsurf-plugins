---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development Commands

```bash
npm run dev          # Start development server on port 3000
npm run build        # Production build
npm run lint         # Run ESLint
npm run typecheck    # TypeScript type checking (tsc --noEmit)
npm run db:migrate   # Run database migrations (tsx scripts/migrate.ts)
npm run user:create  # Create an account (signups are closed by default)
npm run migrate:chat-item-context  # Allow 'item' as a chat context type

# MCP Server
npm run mcp:start    # Start MCP server (requires env vars)
npm run mcp:migrate  # Create mcp_api_keys table
npm run mcp:keygen   # Generate API key: npx tsx scripts/generate-mcp-key.ts <email>

# Testing
npm test             # Run tests once
npm run test:watch   # Run tests in watch mode
npm run test:coverage # Generate coverage report
```

## Key Features

### Command-First Interface
- **UnifiedSearch** (Cmd+K): Primary interaction point for all actions
  - Search notes, tasks, projects, captures
  - Run commands (create, navigate, actions)
  - Keyboard shortcuts: Cmd+Shift+C (Quick Capture), Cmd+Shift+D (Daily Note)
  - Quick Actions: Shown by default when search is empty for discoverability
  - Mobile-optimized: 56px touch targets, full-screen overlay

### Dashboard (`/` → `StreamPage`)

Five bands, ordered so the page answers *"anything for me?"* before *"what have
I got?"*, and **three of them render nothing when they have nothing to say**:

| Band | Component | Renders when |
|------|-----------|--------------|
| Greeting | `stream-page.tsx` | always (one line: greeting, date, "N done") |
| Brain Bar | `brain-bar.tsx` | always — the universal input |
| **Needs you** | `needs-you.tsx` | any of overdue / to review / contacts to sort / follow-ups due / due today / unread notifications is non-zero |
| **People** | `crm-pulse-card.tsx` | the user has contacts; a card when something is pending, one quiet line when not |
| Stream | `stream-feed.tsx` | always |

- **One place announces that something is waiting.** `NeedsYou` is it. The feed
  used to carry its own "N awaiting review" banner counted from the 30 items it
  happened to have loaded, so it disagreed with the sidebar's server-side count.
- **Server-sourced counts.** `getStreamStats` (`src/lib/stream/stats.ts`) and
  `getCrmPulse` (`src/lib/crm/pulse.ts`) run in parallel in the page's server
  component. Stats returns only counters that represent *pending work*; it used
  to return seven, of which the page rendered one. The bell's unread count is
  the sole client fetch, because it changes while the page is open.
- **Filter chips are earned.** A chip renders only for a type the user actually
  has items of, four show at a time, the rest fold behind "+N", and an active
  filter always stays visible. Nine always-on chips was a scrolling wall above
  the content.
- **The legacy dashboard is gone.** `src/components/dashboard/` (hero section,
  conditional sections, insight feeds, widgets — 17 components) was replaced by
  the Stream and had no importer left in the app.

#### Reading the stream: density and time buckets

The feed is a list of short rows at a density the viewer chooses, grouped by
when things happened. One flat chronological list of identical 88px cards showed
about seven items on a 1080p screen and four on a phone.

- **Density** (`src/lib/stream/density.ts`, `useStreamDensity`): `compact`
  (one line) / `cozy` (plus a preview line) / `comfortable` (the original card).
  Rows get shorter by *removing* elements — never by type below 12px or a touch
  target below 44px. At compact the 44px action buttons overflow the 40px row
  through negative margin, so the target is full size while the row stays short.
  Actions are always visible; they used to be `md:group-hover:opacity-100`,
  which made every capability a hover-only secret on the one device that hovers.
- **Buckets** (`src/lib/stream/grouping.ts`, pure and unit-tested): Now (<1h) /
  Earlier today / Yesterday / This week / Earlier, with sticky headings.
  Calendar days, not rolling 24-hour windows — except that recency wins, so
  something written at 23:50 and read at 00:10 is "Now", not "Yesterday".
- **`parseDbTimestamp` is how stream timestamps must be read.** SQLite's
  `datetime('now')` yields `YYYY-MM-DD HH:MM:SS` with no zone marker and is UTC
  by definition; `new Date()` reads that shape as *local*, skewing every
  timestamp by the viewer's offset. Use this, not the `Date` constructor.
- **"Since you were last here"** is a divider drawn at the one point where new
  meets old, and only when both exist. `users` has no `last_seen_at`, so the
  anchor is `localStorage`, read once per page load and immediately overwritten
  — re-reading it while the page is open would make the boundary creep upward
  under the reader.
- **Density and last-visit are `localStorage`, deliberately.** Density is a
  property of the screen, not the account; syncing it would make a phone and a
  desktop fight. Both read through `useSyncExternalStore`, so neither needs a
  `setState` in an effect.
- **`contentWidthClass`** (`src/lib/navigation.ts`) gives the dashboard route

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crawfordind/brain-portal](https://github.com/crawfordind/brain-portal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-19 -->
