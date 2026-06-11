---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FriendScope is a Next.js 15 application that provides scientific friendship assessments through a client-side architecture. The application evaluates friendships across 10 psychological categories using a 20-question assessment, providing users with detailed analytics, historical tracking, and personalized recommendations - all while maintaining complete privacy through local storage.

**Tech Stack:**
- **Framework**: Next.js 15 with App Router, React 19, TypeScript 5
- **Styling**: Tailwind CSS + shadcn/ui components
- **State Management**: Zustand with persist middleware
- **Charts**: Recharts + ApexCharts
- **Animations**: Framer Motion + Lottie React
- **PDF Export**: jsPDF
- **Deployment**: Vercel

## Development Commands

```bash
# Development
npm run dev          # Start dev server at http://localhost:3000
npm run build        # Build for production
npm run start        # Start production server
npm run lint         # Run ESLint

# Package Manager
npm install          # Install dependencies (also: yarn, pnpm)
```

## Architecture

### Core Assessment System

The assessment engine (`lib/assessment.ts`) is the heart of the application:

- **20 questions across 10 categories**: Each question has a weight (1.0-1.5) reflecting psychological importance
- **10 assessment categories**:
  - Trust & Honesty (weight: 1.5) - Core foundation
  - Emotional Support (weight: 1.2)
  - Communication (weight: 1.0)
  - Boundaries (weight: 1.3)
  - Reciprocity (weight: 1.1)
  - Conflict Resolution (weight: 1.2)
  - Growth & Development (weight: 1.0)
  - Values Alignment (weight: 1.4)
  - Respect & Recognition (weight: 1.2)
  - Reliability (weight: 1.3)

- **Scoring algorithm**: Weighted calculation that converts 7-point Likert scale responses into category scores (0-100), then averages to overall score
- **Health assessment thresholds**: Excellent (85+), Good (70-84), Concerning (50-69), Unhealthy (30-49), Toxic (<30)

### State Management Architecture

**Two separate Zustand stores with different purposes:**

1. **`lib/store.ts`** - Assessment State (non-persistent):
   - Manages current assessment session
   - Tracks question progression and answers
   - Resets on completion
   - No persistence - temporary state only

2. **`lib/history-store.ts`** - History State (persistent):
   - Persists completed assessments to localStorage
   - Manages assessment history (max 50 entries)
   - Provides CRUD operations for historical data
   - Key: `friendship-assessment-history`

**Critical distinction**: Assessment state is transient; only completed assessments are saved to history store.

### Data Flow

```
User Input → Assessment Store (temp) → Calculate Scores → Save to History Store (persistent) → Visualization
```

1. User takes assessment (`app/assess/page.tsx`)
2. Answers stored in temporary assessment store (`lib/store.ts`)
3. On completion, scores calculated (`lib/assessment.ts:calculateScores`)
4. Result saved to history store with localStorage persistence (`lib/history-store.ts`)
5. User redirected to results page (`app/results/[id]/page.tsx`)

### Component Structure

```
app/
├── page.tsx                 # Landing page
├── assess/page.tsx          # Assessment interface (stepper through 20 questions)
├── results/[id]/page.tsx    # Individual result detail page
├── results/page.tsx         # All results overview
├── history/page.tsx         # Assessment history with comparison
├── about/page.tsx           # Mission and scientific foundation
├── resources/page.tsx       # Professional support resources
└── layout.tsx              # Root layout with Header/Footer

components/
├── ui/                     # shadcn/ui base components
├── layout/                 # Header.tsx, Footer.tsx
├── dialogs/                # ShareDialog.tsx
├── ComparisonChart.tsx     # Multi-assessment comparison visualization
├── FriendInfoDialog.tsx    # Assessment metadata entry
├── GEOHead.tsx            # SEO/meta tags
├── GEODashboard.tsx       # Analytics dashboard
└── LottieAnimation.tsx     # Animation wrapper

lib/
├── assessment.ts           # Questions, scoring, health assessment logic
├── assessment-utils.ts     # PDF generation, share functionality
├── store.ts               # Assessment state (non-persistent)
├── history-store.ts       # History state (persistent to localStorage)
├── geo-analytics.ts       # Analytics utilities
├── svg-generator.ts       # SVG export functionality
├── fonts.ts               # Font configurations
└── utils.ts               # General utilities (cn, etc.)

types/
└── assessment.ts          # AssessmentResult interface
```

## Key Implementation Details

### Assessment Question Format

Each question in `lib/assessment.ts` follows this structure:
```typescript
{
  id: number,              // 1-20
  text: string,            // Question text
  options: string[],       // 7-point Likert scale (standardOptions)
  category: string,        // One of 10 categories
  weight: number          // 1.0-1.5 (psychological importance)
}
```

### Score Calculation

The `calculateScores` function in `lib/assessment.ts:182`:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChanMeng666/friendscope](https://github.com/ChanMeng666/friendscope) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
