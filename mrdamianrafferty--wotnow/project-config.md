---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains two related applications:

**Go Daisy** - A generalist PWA providing weather-informed recommendations for a wide range of outdoor activities (hiking, cycling, stargazing, etc.). This is the main application with reusable components and patterns for activity recommendations.

**Findr** - A specialist offshoot app focused exclusively on sea fishing predictions. It takes Go Daisy's approach and applies it with deep domain expertise to fishing, using real marine environmental data from CMEMS (Copernicus Marine Environment Monitoring Service) and ICES (International Council for the Exploration of the Sea) rectangular zones.

**Relationship:**
- Go Daisy provides the foundation: location systems, weather integration, UI patterns, translation, user preferences
- Findr reuses and extends these components with fishing-specific logic (species matching, marine data, catch logging)
- Both apps share the same Supabase authentication system (separate routes, shared auth)
- Features developed in Findr (e.g., auto-translation) may be backported to Go Daisy in the future
- Both will be released as separate apps with distinct domains and branding
- The pattern established here will be replicated for other specialist activities in a family of apps

**Domains:**
- Go Daisy: `godaisy.io`
- Findr: `fishfindr.eu`

**Platform Strategy:**
- Currently: Web-only PWAs (both apps)
- Future: iOS and Android native apps once polished (work not yet started)

**Tech Stack:**
- Next.js 15.5 (React 18.3, TypeScript 5.8)
- Supabase (PostgreSQL with RLS policies)
- Tailwind CSS 4 + DaisyUI 5
- TanStack React Query for data fetching
- Framer Motion for animations
- Node 20.x required

## Development Commands

### Core Development
```bash
npm run dev              # Start development server
npm run build            # Production build (includes lint:ci check)
npm run build:strict     # Explicit strict build with linting
npm start                # Start production server
```

### Linting & Type Checking
```bash
npm run lint             # Run ESLint
npm run lint:fix         # Auto-fix ESLint issues
npm run lint:ci          # CI-safe linting (max-warnings=0)
npm run typecheck        # TypeScript type checking without emit
```

### Testing
```bash
npm test                 # Run Jest tests
npm run test:ci          # CI mode with coverage
npm run test:wind        # Run wind recommendation tests
```

### Database & Migrations
```bash
supabase db push         # Apply migrations to Supabase
npm run env:sync         # Sync .env.local to .env.cli for scripts
```

### Deployment
```bash
npm run deploy           # Production deployment script
npm run deploy:quick     # Quick deployment (./quick-deploy.sh)
./vercel-build.sh        # Vercel build script
```

### Scripts & Utilities
```bash
npm run demo:wind        # Demo wind recommendations
npm run demo:soil        # Demo soil conditions
npm run validate:taxonomy # Validate taxonomy data
npm run smoke            # Run smoke tests (bash scripts/smoke-unified.sh)

# Findr-specific seeding
npm run seed:findr:rectangles   # Seed ICES rectangles
npm run seed:findr:conditions   # Seed conditions snapshots
```

## Architecture

### Core Application Structure

**Next.js Pages Router:**
- `pages/` - Route pages (Next.js conventions)
- `pages/api/` - Backend API endpoints
- `pages/index.tsx` - Go Daisy home page
- `pages/weather.tsx` - Main weather dashboard (Go Daisy)
- `pages/activities.tsx` - Activity recommendations (Go Daisy)
- `pages/findr/` - Fishing prediction UI pages (Findr specialist app)
- `pages/_app.tsx` - App wrapper with contexts
- `pages/_document.tsx` - HTML document customization

**Key Directories:**
- `components/` - React components (general + feature-specific)
- `components/findr/` - Findr-specific components (cards, modals, navigation)
- `hooks/` - Custom React hooks
- `lib/` - Shared utilities, services, and business logic
- `context/` - React context providers
- `types/` - TypeScript type definitions
- `data/` - Static data files and lookup tables
- `supabase/migrations/` - Database schema migrations

### Go Daisy Architecture (General Activity Recommendations)

**Core Concept:** Weather-informed activity recommendations for diverse outdoor activities. Components are designed to be reusable across different activity domains.

**Shared Components:**
- Location system (UnifiedLocationContext) - GPS, place search, coordinate management
- Weather services - OpenWeather, Stormglass for marine data
- Translation system - Multi-language support with DeepL API caching
- User preferences - Settings persistence across activities
- UI patterns - Cards, modals, navigation components built with DaisyUI

**Activities Supported:**
- Weather dashboards (current and forecast)
- Astronomy highlights (ISS visibility, moon phases)
- Tide predictions
- Soil conditions
- Wind recommendations
- General outdoor activity suggestions

### Findr Architecture (Fishing Predictions)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mrdamianrafferty) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
