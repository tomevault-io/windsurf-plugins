---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**דאטאבוס (Open Bus Map Search)** is a TypeScript/React web application for visualizing and analyzing Israeli public transportation data. Built by the Public Knowledge Workshop (Hasadna), it queries the Open-Bus API to display bus routes, gaps in service, operator performance, and real-time vehicle locations.

## Development Commands

### Essential Commands

```bash
npm install              # Install dependencies
npm start                # Start dev server on localhost:3000
npm run build            # Build for production (runs TypeScript + Vite)
npm run lint             # Run all linters (TypeScript, ESLint, Stylelint, Prettier)
npm run lint:fix         # Auto-fix linting issues
```

### Testing

```bash
npm test                 # Run all tests (Jest + Playwright, excluding visual)
npm run test:unit        # Run Jest unit tests only
npm run test:unit:ci     # Run Jest with coverage report
npm run test:e2e         # Run Playwright e2e tests
npm run test:e2e:ui      # Run Playwright with interactive UI
npm run test:e2e:visual  # Run visual regression tests (Applitools)
```

### Storybook

```bash
npm run storybook        # Start Storybook on port 6006
npm run build-storybook  # Build static Storybook
```

### Running Single Tests

```bash
# Jest (unit tests)
npx jest path/to/test.test.ts
npx jest --testNamePattern="test name pattern"

# Playwright (e2e tests)
npx playwright test path/to/test.spec.ts
npx playwright test --grep "test name pattern"
```

## Architecture Overview

### Frontend Stack

- **Framework**: React 19 with TypeScript (strict mode)
- **Build Tool**: Vite (using Rolldown)
- **Routing**: React Router v7 with lazy-loaded pages
- **Styling**:
  - Material-UI (MUI) — the target styling system
  - Ant Design, styled-components — legacy, being phased out in favor of MUI
  - SCSS modules
  - RTL support via stylis-plugin-rtl
- **State Management**:
  - @tanstack/react-query for server state (with persistence)
  - React Context for theme and layout state
- **Maps**: Leaflet with react-leaflet and markercluster
- **Charts**: Recharts
- **i18n**: react-i18next (Hebrew, English, Arabic, Russian)

### API Integration

The app communicates with two backend services via `@hasadna/open-bus-api-client`:

1. **Stride API** (`process.env.VITE_STRIDE_API`):
   - GTFS data (routes, stops, schedules)
   - SIRI real-time data (vehicle locations)
   - Aggregations for analytics

2. **Backend API** (`process.env.VITE_BACKEND_API`):
   - Health checks
   - Issue/bug reporting
   - Government transportation data
   - Complaint submissions

API clients are configured in `src/api/apiConfig.ts` and consumed through service modules in `src/api/`.

### Project Structure

```
src/
├── api/                    # API service layer
│   ├── apiConfig.ts        # API client configuration (Stride + Backend)
│   ├── gtfsService.ts      # GTFS data queries (routes, stops)
│   ├── siriService.ts      # Real-time vehicle data
│   ├── gapsService.ts      # Service gap analysis
│   ├── groupByService.ts   # Aggregation helpers
│   ├── serviceDayRoutesService.ts  # Routes for a given service day
│   ├── agencyList.ts       # Operator/agency lookups
│   └── geoService.ts       # Geo helpers
├── pages/                  # Route components (lazy-loaded)
│   ├── dashboard/          # Analytics dashboard with charts
│   ├── gaps/               # Service gap visualization
│   ├── gapsPatterns/       # Gap-patterns analysis
│   ├── historicTimeline/   # Historic timeline view
│   ├── lineProfile/        # Individual line details
│   ├── operator/           # Operator performance
│   ├── singleLineMap/      # Single line on the map
│   ├── timeBasedMap/       # Map with time controls
│   ├── velocityHeatmap/    # Speed heatmap
│   ├── vehicle/            # Vehicle detail view
│   ├── homepage/           # Landing page
│   ├── about/              # About page
│   ├── DataResearch/       # Data research tools
│   ├── bugReport/          # Bug report form
│   ├── publicAppeal/       # Public appeal / complaints
│   ├── hackathon/          # Hackathon event page (temporary)
│   ├── DonateModal/        # Donation modal
│   ├── components/         # Shared page components (timeline, EasterEgg, selectors, …)
│   └── ErrorPage.tsx       # Route-level error boundary
├── layout/                 # App shell
│   ├── index.tsx           # Layout root
│   ├── header/             # Header, language/theme toggles, share button
│   ├── sidebar/            # Sidebar + nav menu + logo
│   ├── ThemeContext.tsx    # Dark/light theme provider (MUI + antd sync)
│   └── LayoutContext.tsx   # Sidebar collapse state
├── routes/                 # React Router configuration
│   ├── index.tsx           # Route definitions with icons (PAGES array)
│   └── MainRoute.tsx       # Main route wrapper
├── hooks/                  # Custom React hooks
│   ├── useVehicleLocations.ts  # React Query hook for live positions
│   └── …                   # useDate, usePageState, useAgencyList, …
├── model/                  # TypeScript domain models (busRoute, busStop, operator, globalState, …)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hasadna/open-bus-map-search](https://github.com/hasadna/open-bus-map-search) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
