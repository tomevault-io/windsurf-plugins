---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**KANVA** is a social media post generator for Swiss sports clubs (Unihockey, Volleyball, and Handball). It automatically creates Instagram-ready posts from sports game data, positioned as "Canva for Sports".

**Tech Stack:**
- React 18 + TypeScript + Vite
- Supabase (PostgreSQL, Auth, Storage, Edge Functions)
- Shadcn/ui components on Tailwind CSS
- TanStack Query for data fetching
- Stripe for subscriptions

## Development Commands

```bash
# Install dependencies
npm install

# Start dev server (runs on http://[::]:8080)
npm run dev

# Build for production
npm run build

# Build for development (with source maps)
npm run build:dev

# Lint code
npm run lint

# Preview production build
npm run preview
```

**Important:** There are no test scripts configured. TypeScript is configured with relaxed rules (`noImplicitAny: false`, `strictNullChecks: false`).

## Architecture

### Core Flow
```
User selects: Sport → Club → Team → Game(s)
  ↓
External Sports APIs (Cloud Functions with GraphQL-like queries)
  ↓
React Query cache
  ↓
Template selection & customization
  ↓
SVG generation with game data
  ↓
Export to PNG (Instagram-optimized: 2160x2700px)
```

### Key Directories

```
src/
├── components/
│   ├── ui/              # Shadcn components (51 files)
│   ├── landing/         # Landing page sections
│   ├── profile/         # User profile & settings
│   └── templates/       # Template designer & management
├── pages/               # Route pages (Index, Studio, Profile, etc.)
├── hooks/               # Custom hooks (useAuth, useSubscription, etc.)
├── contexts/            # React Context providers
├── utils/               # Core utilities (SVG processing)
├── config/              # Configuration (fonts, etc.)
├── integrations/
│   └── supabase/        # Supabase client & generated types
├── translations/        # i18n support (DE/EN)
└── assets/

supabase/
├── functions/           # Edge functions (7 functions)
└── migrations/          # Database migrations (47 files)
```

### Critical Components

- **Studio.tsx** (656 lines): Main app - game selection & post generation
- **GamePreviewDisplay.tsx** (1150+ lines): SVG template preview & PNG export
- **TemplateDesigner.tsx** (2280+ lines): Visual template editor

### Sports Data APIs

Game data comes from **external Cloud Functions**, NOT Supabase:
- **Unihockey**: `myclubmanagement.cloudfunctions.net/api/swissunihockey`
- **Volleyball**: `myclubmanagement.cloudfunctions.net/api/swissvolley`
- **Handball**: `myclubmanagement.cloudfunctions.net/api/swisshandball`

These use GraphQL-like query interfaces.

### Database Schema (Key Tables)

From `src/integrations/supabase/types.ts`:
- **profiles**: User profiles, email preferences, last selections
- **templates**: SVG template configurations (system & custom)
- **user_logos**: Uploaded team/club/sponsor logos
- **user_team_slots**: Teams user follows for notifications
- **subscriptions**: Stripe subscription data
- **subscription_limits**: Feature limits per tier (Free, Amateur, Pro, Premium)

## SVG to PNG Export System

This is the most complex part of the application. **READ THIS CAREFULLY** before modifying export code.

### Two Export Implementations

**1. Legacy: `src/utils/svgToImage.ts` (976 lines)**
- Monolithic approach
- Processes entire SVG at once
- Uses html2canvas for rendering
- Still available as fallback

**2. New (Preferred): `src/utils/svgToImageLayered.ts` (1516 lines)**
- **60% faster, 60% less memory usage, 90% more reliable**
- Separates SVG into 3 layers:
  - Background layer (colors/background images)
  - Images layer (team logos, graphics)
  - Text layer (text + embedded fonts)
- Uses native `createImageBitmap()` instead of html2canvas
- Better font rendering reliability

See `docs/layered-svg-export.md` for full technical documentation.

### Export Process Phases

1. **SVG Preparation**: Clone and offscreen mount SVG
2. **Image Inlining** (20-60%): Convert HTTP URLs to data URLs
   - Uses CORS fetch with Supabase proxy fallback
   - Progress tracking per image
3. **Font Management** (60-70%): Embed custom fonts
   - Fonts configured in `src/config/fonts.ts` (Bebas Neue, Roboto, Open Sans, Lato, Montserrat)
   - WOFF2 format converted to data URLs
   - Three embedding methods for maximum compatibility
4. **Canvas Rendering** (70-90%): Convert layers to canvas
5. **Finalization** (90-100%): Export as PNG/JPEG/WebP

### Font Handling

All fonts are centrally configured in `src/config/fonts.ts`:
```typescript
export const AVAILABLE_FONTS: Record<string, FontConfig> = {
  'bebas-neue': {
    displayName: 'Bebas Neue',
    cssFamily: 'Bebas Neue',
    googleFontsUrl: '...',
    variants: [{ weight: '400', style: 'normal', url: '...' }]
  },
  // ... more fonts
}
```

**Important:** Font names must match exactly between SVG `font-family` attributes and `cssFamily` in config.

### Platform-Specific Export

- **Mobile**: Uses native Share API (Capacitor)
- **Desktop**: Direct download
- **Instagram optimization**: 1080x1350 base @ 2x scale = 2160x2700px

## Subscription & Limits


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [myclubapp/myclub-fanpost](https://github.com/myclubapp/myclub-fanpost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
