---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CivicStream is a Next.js 14 demo application showcasing a Canadian civic engagement platform. It unifies Federal, Provincial, and Municipal government data into a single mobile-first feed. This is a **demo build** using fixture data throughout — designed for investor presentations.

## Development Commands

```bash
# Install dependencies
pnpm install

# Run development server
pnpm dev

# Build for production
pnpm build

# Start production server
pnpm start

# Lint code
pnpm lint
```

## Architecture Overview

### Tech Stack
- **Framework**: Next.js 14 with App Router
- **Language**: TypeScript (strict mode)
- **Styling**: Tailwind CSS
- **State Management**: React Context + useReducer (no external state library)
- **Mapping**: Leaflet.js + react-leaflet (dynamically imported to avoid SSR issues)
- **Animations**: Framer Motion for page transitions and micro-interactions
- **Icons**: Lucide React
- **Data Source**: Local JSON fixtures in `/data`

### State Management Pattern

All global state is managed through `context/CivicStreamContext.tsx`:
- Uses `useReducer` for predictable state updates
- Persists to `localStorage` on the client
- Initial state seeds demo data (10-day streak, 3 earned badges, 2 read bills)

**Important**: State is reset when user clicks "Reset Demo" on profile page.

### Data Flow

```
data/*.json (fixtures)
  ↓
lib/hooks/useDemoData.ts (DEMO_MODE = true)
  ↓
Components (via hooks)
```

**Production Upgrade Path**: Each hook file has a comment block showing exactly how to replace fixture imports with real API calls.

## Key Architectural Decisions

### 1. Demo Mode vs Production
- `DEMO_MODE` constant in `lib/hooks/useDemoData.ts` controls data source
- All data-fetching hooks include upgrade path comments
- When `DEMO_MODE = false`, hooks should attempt API fetches (gracefully handle failures)

### 2. Leaflet Dynamic Import
The Zoning Map uses `next/dynamic` with `ssr: false` to prevent hydration errors:

```typescript
const ZoningMap = dynamic(
  () => import("@/components/map/ZoningMap").then((mod) => ({ default: mod.ZoningMap })),
  { ssr: false }
);
```

**Never** import Leaflet components directly without this wrapper.

### 3. Jurisdiction Color System
Three government levels have dedicated colors defined in `lib/utils/jurisdiction.ts`:
- Federal: #D71920 (red)
- Provincial: #003F8A (blue)
- Municipal: #2D7A45 (green)

Use helper functions for consistent styling:
- `getJurisdictionColor()` - Hex color
- `getJurisdictionBgClass()` - Tailwind classes
- `getJurisdictionBorderClass()` - Border classes

### 4. Type Safety
All TypeScript interfaces are defined in `types/index.ts`. **Never use `any`** in components or hooks. All fixture data is strongly typed against these interfaces.

## Common Development Tasks

### Adding a New Bill
1. Add entry to `data/bills.json` with all required fields
2. Type will be validated against `Bill` interface
3. Bill auto-appears in feed if it has matching `linkedBillId` in `data/feed.json`

### Adding a New Badge
1. Add to `data/badges.json`
2. Use a valid Lucide icon name in `icon` field
3. `BadgeGrid` component dynamically renders the icon
4. To mark as earned, add badge `id` to `earnedBadgeIds` in `CivicStreamContext.tsx` initial state

### Modifying Feed Items
Feed items in `data/feed.json` support these types:
- `bill` - Links to `/bill/[id]`
- `meeting` - Generic council meeting
- `petition` - Community petition
- `alert` - Zoning or infrastructure alert (links to `/zoning`)
- `lobbywatch` - Lobby disclosure item
- `budget` - Budget announcement

Set `isNew: true` for items to show NEW badge. Set `urgency: "high"` for urgent styling.

### Creating New Pages
All pages are in `app/` directory using App Router conventions:
- Layout includes `<TopBar>` and `<BottomNav>` (hidden on onboarding)
- Wrap content in `max-w-md mx-auto` for mobile-first centered layout
- Use `"use client"` directive when using React hooks or interactivity

## Important Patterns

### Date Formatting
Import from `lib/utils/formatDate.ts`:
- `formatRelativeDate()` - "2 hours ago", "Yesterday", etc.
- `formatFullDate()` - "January 15, 2025"

### Navigation
Use Next.js `useRouter()` hook:
```typescript
import { useRouter } from "next/navigation";
const router = useRouter();
router.push("/feed");
```

### Accessing Global State
```typescript
import { useCivicStream } from "@/context/CivicStreamContext";
const { state, dispatch } = useCivicStream();
```

Dispatch actions:
- `SET_POSTAL_CODE` - Update postal code
- `COMPLETE_ONBOARDING` - Mark onboarding done
- `MARK_BILL_READ` - Add bill to read list
- `SET_FILTER` - Change feed filter
- `INCREMENT_STREAK` - Increment day counter
- `RESET_DEMO` - Reset to initial state

## File Organization

### Components
Group by feature in subdirectories:
- `components/nav/` - Navigation (TopBar, BottomNav)
- `components/feed/` - Feed items and filters
- `components/bill/` - Bill detail sections
- `components/reps/` - Representative cards
- `components/map/` - Zoning map and alerts
- `components/gamification/` - Badges, streaks, activity

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/faizaanqureshi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
