---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a monorepo containing:
1. **Educational Documentation** (`/docs`, `README.md`) - Comprehensive guide to GitHub profile enhancement, ranking systems (GitHub Stats & Trophies), and achievements
2. **Code Warrior Application** (`/code-warrior`) - Next.js web app that gamifies GitHub contributions into an RPG experience

## Development Commands

### Code Warrior Application

All commands should be run from the `code-warrior/` directory.

**Setup:**
```bash
cd code-warrior
npm install
```

**Development:**
```bash
npm run dev          # Start Next.js dev server on http://localhost:3000
```

**Production:**
```bash
npm run build        # Build for production
npm start            # Run production server
```

**Linting:**

```bash
npm run lint         # Run ESLint
```

**Database Management:**

```bash
npm run db:types     # Generate Supabase TypeScript types (from schema)
npm run db:push      # Push schema changes to Supabase
npm run db:pull      # Pull remote schema from Supabase
npm run db:migration # Create a new migration file
npm run db:status    # Check database connection status
```

## Code Warrior Architecture

### Tech Stack
- **Framework:** Next.js 16.1.1 with App Router
- **Language:** TypeScript 5 (strict mode)
- **Styling:** Tailwind CSS v4
- **Database:** Supabase (PostgreSQL)
- **Auth:** NextAuth.js 4.24.13 with GitHub OAuth
- **State Management:** TanStack Query (React Query) v5
- **Animations:** Framer Motion v12
- **UI/Utilities:** Lucide React (icons), dnd-kit (drag-and-drop), html2canvas (export)

### Core Architecture Patterns
refer to docs/architecture.md

#### 1. Sync Engine Pattern
The application uses a "Sync Engine" to minimize GitHub API calls:
- User triggers sync via POST `/api/sync`
- Sync service fetches GitHub data (profile, repos, events, PRs, issues)
- Data is processed to calculate XP, rank, and RPG stats
- Results are cached in Supabase
- Frontend reads from Supabase (not GitHub API directly)
- **Cooldown:** 5 minutes between syncs to prevent rate limit abuse

**Key Files:**
- `src/app/api/sync/route.ts` - Sync Engine API endpoint
- `src/lib/github.ts` - GitHub API wrapper with caching
- `src/lib/game-logic.ts` - XP and rank calculations

#### 2. XP Calculation System

GitHub stats are converted to XP using weighted formulas:
```typescript
XP_WEIGHTS = {
  STAR: 50,        // Stars = Charisma/Fame
  PR: 40,          // Pull Requests = Strength
  COMMIT: 10,      // Commits = Stamina/Health
  ISSUE: 15,       // Issues = Wisdom
  REVIEW: 20,      // Reviews = Wisdom
}
```

**Rank Progression:**
- C (Novice): 0-999 XP
- B (Intermediate): 1,000-2,999 XP
- A (Skilled): 3,000-5,999 XP
- AA (Advanced): 6,000-9,999 XP
- AAA (Elite): 10,000-14,999 XP
- S (Expert): 15,000-24,999 XP
- SS (Master): 25,000-49,999 XP
- SSS (Legend): 50,000+ XP

**Implementation:** `src/lib/game-logic.ts`

#### 3. RPG Stats Mapping

GitHub metrics map to RPG attributes:
- **Health (HP):** Based on commits (consistency)
- **Mana (MP):** Based on issues + reviews
- **Strength:** Based on pull requests
- **Charisma:** Based on stars received
- **Wisdom:** Based on issues + reviews

All stats are capped at 100.

#### 4. Quest System

Quests track specific GitHub milestones:
- Defined in `quests` table (title, description, criteria_type, criteria_threshold, xp_reward)
- User progress tracked in `user_quests` join table
- Criteria types: `repo_created`, `pr_merged`, `commits`, `stars_received`, `issues_created`
- Quest completion checked during sync via `src/lib/quest-logic.ts`

**API Endpoints:**
- GET `/api/quests` - Fetch all quests with user progress
- POST `/api/quests/claim` - Claim quest reward

#### 5. Badge System

Badges are collectible achievements that provide stat boosts:
- Defined in `badges` table (name, icon_slug, stat_boost as JSONB)
- Users can equip/unequip badges for active bonuses
- Badge inventory tracked per user
- Badges can be earned through quest rewards or direct completion

**API Endpoints:**
- GET `/api/badges/inventory` - Fetch user's collected badges
- POST `/api/badges/equip` - Equip a badge (adds stat bonuses)
- POST `/api/badges/unequip` - Unequip a badge (removes bonuses)

#### 6. Leaderboard System

Tracks top players globally based on XP and rank:
- Real-time leaderboard with pagination
- Displays username, rank tier, XP, and achievement stats

**API Endpoint:**
- GET `/api/leaderboard` - Fetch top players with stats

#### 7. Additional Features

##### Drag-and-Drop Badge Management (dnd-kit)

- Users can reorganize equipped badges
- Visual feedback during drag operations
- Persistent slot assignments

##### Profile Export (html2canvas)

- Shareable character card as image
- Export stats and achievements
- Social media friendly format

##### Onboarding Tutorial

- First-time user guidance
- Game mechanics explanation
- Quick sync demonstration

##### Notifications System

- Toast notifications for sync events
- Quest completion alerts
- XP gain feedback
- Achievement unlocks

##### Performance Optimizations

- usePerformanceMode hook for low-end devices

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [smirk-dev/Guide-to-levelling-up-on-Github](https://github.com/smirk-dev/Guide-to-levelling-up-on-Github) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
