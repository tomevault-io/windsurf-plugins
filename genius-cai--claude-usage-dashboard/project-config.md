---
trigger: always_on
description: A modern web dashboard for monitoring Claude Code usage, built with Next.js 16 (App Router) and FastAPI. Provides real-time usage tracking, cost analysis, and visual statistics for Claude AI API consumption.
---

# Claude Usage Dashboard

A modern web dashboard for monitoring Claude Code usage, built with Next.js 16 (App Router) and FastAPI. Provides real-time usage tracking, cost analysis, and visual statistics for Claude AI API consumption.

**GitHub**: https://github.com/Genius-Cai/claude-usage-dashboard

## Quick Reference

### Essential Commands
```bash
# Frontend
npm run dev          # Start dev server (localhost:3000)
npm run build        # Production build
npm run lint         # ESLint check
npm run test         # Run tests in watch mode
npm run test:run     # Run tests once
npm run test:coverage # Run with coverage

# Backend (from /backend)
pip install -e .     # Install with dependencies
uvicorn app.main:app --reload --port 8000
cd backend && pytest  # Run backend tests

# Both services
make dev             # Start both servers
make docker-dev      # Docker development
```

### API Endpoints Quick Reference
| Endpoint | Description |
|----------|-------------|
| `GET /api/usage/realtime` | Real-time usage with session info |
| `GET /api/usage/plan-usage?plan=max20` | Plan usage vs limits |
| `GET /api/usage/daily?date=YYYY-MM-DD` | Daily statistics |
| `GET /api/stats/models?days=30` | Usage by model |
| `WS /ws/realtime` | WebSocket live updates |
| `GET /health` | Health check |

## Current Status (Dec 2024)

### Working Features
- **Dashboard Home**: Real-time stats, usage trends, model distribution, token breakdown
- **Plan Usage Card**: Accurate data from `analyze_usage()` matching CLI output
- **Session Timer**: Countdown with progress bar using plan-usage API
- **Statistics Page**: Historical data with date range selection and CSV export
- **Settings Page**: Plan, currency, timezone, theme, notifications configuration
- **WebSocket**: Real-time updates with exponential backoff reconnection
- **PWA**: Installable on iOS/Android
- **Performance**: Code splitting, conditional data fetching, cache warming

### Recent Optimizations (Dec 2024)
- Tab components loaded dynamically with `next/dynamic` (code splitting)
- Data fetching only occurs when tabs are active
- Backend cache warming at startup for faster first requests
- WebSocket exponential backoff reconnection (1s → 2s → 4s → ... → 30s cap)

### Known Issues
- Sessions page could benefit from performance optimization
- Project statistics endpoint returns empty (needs project path extraction)

## Tech Stack

### Frontend
- **Framework**: Next.js 16 (App Router) with TypeScript
- **UI Components**: shadcn/ui (Radix UI primitives)
- **Styling**: Tailwind CSS v4
- **Charts**: Recharts
- **State Management**: Zustand (persisted to localStorage)
- **Data Fetching**: TanStack Query v5 (30-60 second cache)
- **Animations**: Framer Motion
- **PWA**: next-pwa
- **Theme**: next-themes (system/light/dark)

### Backend
- **Framework**: FastAPI (Python 3.10+)
- **Data Source**: claude-monitor v3.1.0+ package
- **Session Calculation**: Uses `analyze_usage()` for proper session blocks
- **Real-time**: WebSocket with ConnectionManager
- **Caching**: In-memory cache with 10-second TTL

### Deployment
- Docker Compose with Nginx reverse proxy
- Tailscale/Cloudflare tunnel support
- Data sync scripts for Mac → Server

## Project Structure

```
claude-usage-dashboard/
├── src/                          # Frontend source
│   ├── app/                      # Next.js App Router pages
│   │   ├── layout.tsx            # Root layout with providers
│   │   ├── page.tsx              # Dashboard (Overview/Analytics/Models tabs)
│   │   ├── sessions/page.tsx     # Session management
│   │   ├── settings/page.tsx     # User preferences
│   │   └── statistics/page.tsx   # Historical analytics
│   ├── components/
│   │   ├── ui/                   # shadcn/ui (Button, Card, Tabs, etc.)
│   │   ├── charts/               # UsageTrendChart, ModelDistribution, TokenBreakdown
│   │   ├── dashboard/            # StatsCard, SessionTimer, PlanUsageCard, PlanSelector
│   │   ├── layout/               # AppLayout, Sidebar, Header, MobileNav
│   │   └── providers/            # ThemeProvider, QueryProvider
│   ├── hooks/
│   │   ├── use-usage-data.ts     # TanStack Query hooks for all data
│   │   └── use-websocket.ts      # WebSocket connection management
│   ├── stores/
│   │   └── settings-store.ts     # Zustand store (plan, currency, theme, etc.)
│   ├── lib/
│   │   ├── api.ts                # API client with retry logic
│   │   └── utils.ts              # cn() helper
│   └── types/
│       └── index.ts              # All TypeScript types (40+)
├── backend/
│   └── app/
│       ├── main.py               # FastAPI app with lifespan
│       ├── core/config.py        # Pydantic Settings
│       ├── routers/
│       │   ├── usage.py          # /api/usage/* endpoints
│       │   ├── stats.py          # /api/stats/* endpoints
│       │   └── websocket.py      # /ws/realtime WebSocket
│       ├── services/
│       │   └── data_service.py   # Bridge to claude-monitor
│       └── models/
│           └── schemas.py        # Pydantic response models
├── docker/                       # Docker Compose files

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Genius-Cai/claude-usage-dashboard](https://github.com/Genius-Cai/claude-usage-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
