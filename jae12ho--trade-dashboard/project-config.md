---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a real-time financial market dashboard built with Next.js 16 (App Router) + React 19 + TypeScript. It aggregates 11 economic indicators from multiple external APIs (FRED, Yahoo Finance, CoinGecko) and provides AI-powered market analysis in Korean using Google Gemini.

**Key Technologies:**
- Next.js 16.1.1 with App Router (file-based routing)
- React 19.2.3 with client/server component split
- TypeScript 5 (strict mode)
- Tailwind CSS 4 (with dark mode support)
- Google Gemini API (gemini-2.5-flash) for AI analysis
- Recharts 3.6.0 for data visualization

## Development Commands

```bash
# Start development server (http://localhost:3000)
npm run dev

# Build production bundle
npm run build

# Run production server
npm start

# Run ESLint type checking
npm run lint
```

## Environment Setup

Create `.env.local` in project root with required API keys:

```bash
GEMINI_API_KEY=<your-key>              # https://makersuite.google.com/app/apikey
FRED_API_KEY=<your-key>                # https://fred.stlouisfed.org/docs/api/api_key.html
UPSTASH_REDIS_REST_URL=<your-url>      # https://console.upstash.com (Redis database)
UPSTASH_REDIS_REST_TOKEN=<your-token>  # REST API credentials from Upstash
```

Yahoo Finance and CoinGecko APIs require no authentication.

## Architecture Overview

### Data Flow Pattern

```
Client (Dashboard.tsx)
  ↓ fetch('/api/indicators')
Server API Route (/app/api/indicators/route.ts)
  ↓ getAllIndicators()
Indicator Fetch Functions (lib/api/indicators.ts)
  ↓ parallel Promise.all() for 11 indicators
External APIs (FRED, Yahoo Finance, CoinGecko)
  ↓ raw data responses
Data Transformation & Normalization
  ↓ IndicatorData with current/previous/history
Return to Client
  ↓ render in 3x3 grid
IndicatorCard + MiniChart components
```

### Key Design Patterns

1. **Server/Client Split**: Pages are server components, Dashboard is client component (`'use client'`)
2. **Adapter Factory**: Each external API has dedicated fetch function that returns normalized `{ current, previous, history }` format
3. **Parallel Aggregation**: `getAllIndicators()` uses `Promise.all()` to fetch 11 indicators concurrently
4. **Polling Pattern**: Dashboard auto-refreshes every 5 minutes via `setInterval`
5. **Force Dynamic**: API routes export `dynamic = 'force-dynamic'` to prevent caching

### Directory Structure

```
/app
  page.tsx                    # Home page (renders Dashboard)
  layout.tsx                  # Root layout with fonts & metadata
  globals.css                 # Tailwind imports + CSS variables + wiggle animation
  /api
    /indicators
      route.ts                # GET endpoint - returns all 11 indicators
    /ai-prediction
      route.ts                # GET endpoint - returns Gemini market analysis
    /indicator-comments
      route.ts                # POST endpoint - returns AI comments for indicators

/components
  Dashboard.tsx               # Main client component (state, fetching, layout)
  IndicatorCard.tsx           # Individual metric card display with AI comment
  MiniChart.tsx               # Recharts line chart for 30-day trend
  AIPrediction.tsx            # AI sentiment & analysis display
  ThemeScript.tsx             # Dark mode initialization script

/lib
  /types
    indicators.ts             # TypeScript interfaces (single source of truth)
    errors.ts                 # Custom error types (QuotaError, isQuotaError, createQuotaError)
  /constants
    gemini-models.ts          # Gemini model names & DEFAULT_GEMINI_MODEL (single source of truth)
  /api
    indicators.ts             # External API fetch functions + generateAIComments
    gemini.ts                 # Google Gemini API integration (market + comments)
  /cache
    gemini-cache-redis.ts     # Market analysis cache (24h TTL)
    indicator-comment-cache.ts # Individual indicator comment cache (24h TTL)

/ai
  PLAN.md                     # Development plans (Phase 7, 8, 9)
  TO_DO.md                    # Task tracking
```

## The 11 Indicators

**Macro Indicators (6):**
1. US 10Y Yield - FRED: `DGS10`
2. US Dollar Index (DXY) - Yahoo Finance: `DX-Y.NYB`
3. High Yield Spread - FRED: `BAMLH0A0HYM2`
4. M2 Money Supply - FRED: `M2SL`
5. Consumer Price Index (CPI) - FRED: `CPIAUCSL` (monthly)
6. Total Nonfarm Employment - FRED: `PAYEMS` (monthly)
   - Note: Displays total employment level (e.g., 159.53M), not monthly change
   - 1M change shows monthly job creation/loss (e.g., +0.05M = 50K jobs added)

**Commodity & Asset Indicators (3):**
7. Crude Oil (WTI) - Yahoo Finance: `CL=F`
8. Copper/Gold Ratio - Yahoo Finance calculated: `HG=F / GC=F × 10000`
9. Bitcoin (BTC/USD) - CoinGecko: `bitcoin`

**Market Sentiment Indicators (2):**
10. Manufacturing Confidence - FRED: `BSCICP02USM460S` (OECD)
11. VIX (Fear Index) - Yahoo Finance: `^VIX`

## Adding New Indicators

To add a new indicator, follow this pattern:

1. **Add TypeScript interface** (if new API source):
   - Edit `lib/types/indicators.ts`
   - Add response type interface (e.g., `NewAPIResponse`)

2. **Create fetch function** in `lib/api/indicators.ts`:
   ```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jae12ho/trade-dashboard](https://github.com/Jae12ho/trade-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
