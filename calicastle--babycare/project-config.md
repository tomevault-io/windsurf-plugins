---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
pnpm dev          # Start dev server (Vite)
pnpm build        # Type-check (tsc -b) then build for production
pnpm lint         # ESLint
pnpm preview      # Preview production build locally
```

## Tech Stack

- **React 19** + **TypeScript 5.9** (strict mode) with **Vite 7**
- **Tailwind CSS 4** via `@tailwindcss/vite` plugin (no postcss config needed)
- **React Router DOM 7** with hash-based routing (`HashRouter` in `main.tsx`)
- **@base-ui/react** — headless UI components (Tabs, Collapsible, Dialog, AlertDialog, Progress, NumberField, Toggle, ToggleGroup, ScrollArea)
- **react-day-picker** — date picker (zh-CN locale, used in bottom sheet Dialog)
- **sileo** — toast notifications (`sileo.success()`, `sileo.error()`, `sileo.info()`)
- **Dexie.js 4** (IndexedDB) for persistent data, **localStorage** for settings
- **vite-plugin-pwa** for offline-first PWA with Workbox caching
- **Nucleo icons** — `nucleo-glass` (dock nav + action button), `nucleo-ui-outline-duo-18` / `nucleo-ui-fill-duo-18` (feature tool icons)
- **pnpm** as package manager

## Architecture

This is 宝宝助手 (BabyCare) — a Chinese-only pregnancy/baby care PWA with a Duolingo-inspired UI. All UI text is hardcoded in Chinese. No backend, no auth — everything is local to the device.

### Routing

Routes are defined in `src/App.tsx`. Two kinds:

- **Layout routes** (wrapped in `<Layout>` with Dock + ScrollArea): `/`, `/history`, `/settings`, `/tools/*` home pages
- **Full-screen session routes** (no Dock, no Layout): `/tools/kick-counter/session`, `/tools/contraction-timer/session/:sessionId`, `/tools/feeding-log/session/:recordId`

When navigating away from sessions, use `navigate(path, { replace: true })` to prevent back-button confusion.

### Data Layer

- **Dexie database** (`src/lib/db.ts`): `KickCounterDB` with tables `sessions`, `contractionSessions`, `contractions`, `hospitalBagItems`, `feedingRecords`. Schema versions are incremental — always add a new `db.version(N+1)` when adding tables/indexes.
- **Settings** (`src/lib/settings.ts`): stored in localStorage under `babycare-settings`. Includes `goalCount`, `mergeWindowMinutes`, `colorMode`, `dueDate`. Also exports helpers `getDaysUntilDue()` and `getWeeksPregnant()`.

### File Organization

- `src/components/` — reusable UI components (Layout, Dock, StickyHeader, ProgressRing, TipBanner)
- `src/hooks/` — custom React hooks (useDockGesture)
- `src/lib/` — pure utilities (db, settings, time, haptics, tips, encouragements, tools, feeding-helpers, hospital-bag-presets)
- `src/pages/` — route-level components
- `src/pages/tools/<feature>/` — each tool gets its own subdirectory (kick-counter, contraction-timer, hospital-bag, feeding-log)

### Key Conventions

- Components are **default exports**: `export default function ComponentName()`
- Hooks are **named exports**: `export function useHookName()`
- Imports use **explicit `.ts`/`.tsx` extensions**
- Timestamps are **milliseconds** (`Date.now()`), IDs are **`crypto.randomUUID()`**
- No state management library — React hooks + local component state only
- Color mode via `.dark` class toggle on `<html>` — supports system/light/dark (see `applyColorMode()`)
- Use `sileo.success()` / `sileo.error()` / `sileo.info()` for user feedback instead of `alert()`

### Design System

Duolingo-inspired, flat, clean, and playful. Defined in `src/index.css` and applied via Tailwind utility classes throughout.

#### Core Principles

1. **No shadows** — Never use `shadow-*` classes on cards, containers, or buttons.
2. **Border-based contrast** — Use soft borders to define card edges: `border border-gray-200 dark:border-gray-700/60`.
3. **Bold typography** — Headlines use `font-extrabold`, labels use `font-bold`.
4. **Flat color fills** — Buttons and accents use solid Duo palette colors, never gradients (except the hero banner).

#### Color Palette (`src/index.css`)

| Token              | Hex       | Usage                                   |
|--------------------|-----------|------------------------------------------|
| `duo-green`        | `#58CC02` | Primary actions, kick counter, active toggle states |
| `duo-green-dark`   | `#46a302` | Button bottom borders (pressed look)     |
| `duo-orange`       | `#FF9600` | Streaks, contraction timer, warnings     |
| `duo-blue`         | `#1CB0F6` | Kick counter icon, informational accents |
| `duo-purple`       | `#CE82FF` | Due date, date picker accent             |
| `duo-red`          | `#FF4B4B` | Danger actions, stop buttons, alerts     |
| `duo-yellow`       | `#FFC800` | Celebrations, highlights                 |
| `duo-gray`         | `#E5E5E5` | Disabled states, separators              |

#### CSS Variables (`src/index.css`)

| Variable | Light | Dark | Usage |
|----------|-------|------|-------|
| `--sileo-fill` | `#f3f3f3` | `#161616` | Toast notification SVG fill |
| `--dock-accent-1` | `#1a1a1a` | `#ffffff` | Nucleo glass icon gradient stop 1 |
| `--dock-accent-2` | `#404040` | `#d4d4d4` | Nucleo glass icon gradient stop 2 |

#### Cards

- Background: `bg-white dark:bg-[#16213e]`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CaliCastle/babycare](https://github.com/CaliCastle/babycare) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
