---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Capibarismo** is a civic engagement platform for Peru's presidential elections that gamifies candidate comparison using a 90s fighting game aesthetic. The core experience is a **single-elimination tournament** where 36 candidates are narrowed down to a personal top 3 through pick-one-from-three and 1v1 matchups. Users also explore detailed candidate information, compare candidates side-by-side, and view a political compass.

**Core Philosophy: "Game Feel"**
- **The Punch** (<100ms): Voting must feel instant and visceral (requires optimistic UI updates)
- **The Flow** (<1s): Transitions must maintain user engagement without interruption
- **The Reach**: Optimized for 3G networks in rural Peru (high latency tolerance)

## Common Commands

### Development
```bash
npm install              # Install dependencies
npm run dev              # Start dev server at http://localhost:8080
npm run build            # Production build (validates data, generates sitemap)
npm run build:dev        # Development build (for testing)
npm run preview          # Preview production build locally
```

### Code Quality
```bash
npm run lint             # Run ESLint
npm run typecheck        # TypeScript type checking (no emit)
npm test                 # Run Vitest tests
npm test:watch           # Watch mode for tests
npm test:coverage        # Generate coverage report
npm test:ui              # Visual test UI
```

### Load Testing
```bash
npm run loadtest:smoke           # Quick test (5 users, 1 min)
npm run loadtest:baseline        # Normal load (10-50 users, 5 min)
npm run loadtest:peak            # Peak traffic simulation (1k users, 15 min)
npm run loadtest:peru            # Peru-specific network profiles

# Test specific network conditions
NETWORK_PROFILE=rural npm run loadtest:peru      # 3G rural
NETWORK_PROFILE=urban npm run loadtest:peru      # 4G urban
NETWORK_PROFILE=congested npm run loadtest:peru  # Peak hours
```

### Data & Validation
```bash
npm run generate:sitemap     # Generate sitemap.xml
tsx scripts/validate-data.ts # Validate candidate data structure
```

## Architecture Overview

### Frontend Stack
- **React 18** + **TypeScript** + **Vite** (with SWC for fast refresh)
- **Routing**: React Router with lazy-loaded pages
- **Animations**: Framer Motion for overlays, transitions, and bracket zoom
- **State Management**:
  - Zustand for tournament state (`useTournamentStore`) and UI state (`useGameUIStore`, `useCompareStore`)
  - TanStack Query for API calls (ranking page, with aggressive caching)
  - localStorage persistence via Zustand `persist` middleware (tournament state)
- **Styling**: Tailwind CSS + shadcn/ui components
- **Analytics**: PostHog (optional, graceful degradation if not configured)

### Backend (Vercel Serverless)
Located in `/api/` directory:
- **POST /api/game/vote** - Records user votes (fire-and-forget, optimized for write throughput)
- **GET /api/ranking/personal** - Calculates personalized rankings from vote history
- **Storage**: Dual adapter pattern (in-memory for dev, Vercel Blob for production)

### Key Directories
```
/api/                  # Vercel serverless functions
/src/
  /pages/              # Route-level components (lazy-loaded)
  /components/         # Feature-organized UI components
    /game/             # Core game UI (VSScreen, GameHUD, OnboardingModal, CandidateInfoOverlay)
    /tournament/       # Tournament bracket system (BracketTreePage, BracketTree, PickFromThree, PodiumScreen)
    /compare/          # Side-by-side comparison tool
    /political-compass/ # 2D political visualization
    /ui/               # shadcn/ui design system primitives
  /hooks/              # Custom React hooks (game API, legacy Elo hooks)
  /store/              # Zustand state stores (tournament, game UI, compare)
  /services/           # Business logic (tournament logic, session management)
  /data/               # Candidate data and type definitions
    /domains/          # Structured candidate information (education, income, etc.)
  /lib/                # Utilities, constants, types (tournamentTypes, tournamentConstants)
/scripts/              # Build and validation scripts
/load-tests/           # k6 performance test scenarios
```

## Critical Architectural Patterns

### 1. Tournament State Machine (Primary Game Flow)
**Location**: `src/services/tournamentService.ts`, `src/store/useTournamentStore.ts`, `src/lib/tournamentTypes.ts`

The core game is a single-elimination tournament with 36 candidates and 19 total decisions:

**Tournament format:**
| Round | Type | Matches | Description |
|-------|------|---------|-------------|
| R0 | pick-one-from-three | 12 | 36 → 12 candidates |
| R1 | pick-one-from-three | 4 | 12 → 4 candidates |
| R2 | 1v1 semifinal | 2 | 4 → 2 candidates |
| R3 | 1v1 final | 1 | 2 → 1 winner |

**Phase state machine:**
```
null → onboarding → bracket-preview → playing-pick-three/playing-1v1
  → round-transition → (next playing phase) → ... → podium
```

**Key design decisions:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SanGoku95/capibarismo](https://github.com/SanGoku95/capibarismo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
