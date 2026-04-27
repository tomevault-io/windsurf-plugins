---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

BGS Sensor Network Dashboard built with Next.js 15.4.5, TypeScript, and shadcn/ui components. Provides real-time monitoring of BGS's environmental sensor network using the BGS FROST API.

## Commands

- `npm run dev --turbopack` - Start development server with turbopack (fast refresh)
- `npm run dev -- -p 4000` - Start development server on port 4000  
- `npm run build` - Build for production
- `npm run start` - Start production server
- `npm run lint` - Run ESLint

## Architecture

### Tech Stack
- **Next.js 15.4.5** with App Router
- **React 19** with TypeScript 5
- **shadcn/ui** components with stone theme
- **Tailwind CSS v4**
- **BGS FROST API** for sensor data

### Key Components
```
src/
├── app/
│   └── sensor/[sensorId]/page.tsx     # Full sensor page with charts
├── components/
│   ├── ui/                            # shadcn/ui components
│   └── dashboard/
│       ├── BGSDashboard.tsx           # Main dashboard
│       ├── SensorTable.tsx            # Sensor list with filters
│       ├── SensorDetailSheet.tsx      # Sidebar with datastream details
│       ├── DatastreamChart.tsx        # Chart component (normalised by default)
│       └── DatastreamSummary.tsx      # Statistics cards
├── lib/
│   ├── api-config.ts                  # API base URL configuration
│   ├── bgs-api.ts                     # BGS FROST API functions
│   ├── chart-utils.ts                 # Simple chart utilities
│   └── date-utils.ts                  # Date formatting utilities
└── types/
    └── bgs-sensor.ts                  # TypeScript interfaces
```

## Key Features

### Dashboard
- **Sensor Table** - All sensors with search/filter by type, location, measurement
- **Summary Cards** - Total sensors, locations, sites, datastreams
- **Manual Refresh** - User-controlled data updates with visual feedback

### Chart System (Latest - Production Ready)
- **500 Latest Readings Default** - Shows most recent data on first load (no date constraints)
- **Normalised by Default** - 0-1 scaling for comparing different value ranges
- **Raw/Normalised Toggle** - Switch between actual values and normalised view
- **15 Unique Colours** - No colour conflicts between datastreams
- **Adaptive Y-Axis** - 5% padding for better min/max value readability
- **Smart Time Display** - X-axis shows time + date based on data range
- **Dynamic Date Inputs** - Update to reflect actual data range being displayed
- **Any Number of Datastreams** - No hardcoded limits

### Sensor Details
- **Full Sensor Pages** - `/sensor/[sensorId]` with URL routing
- **Interactive Date Pickers** - Auto-update to show actual data range
- **500 Latest Readings** - Default view shows most recent sensor data
- **Custom Date Ranges** - Up to 1000 readings for specific time periods
- **Location & Maps** - GPS coordinates with Leaflet integration
- **Depth Information** - Z-coordinate display showing sensor depth/elevation with coordinate reference system
- **Detail Sheets** - Sidebar with:
  - Chart visualization
  - Summary statistics
  - **Clickable datastream list** - Select to see latest 10 observations
  - Borehole reference extraction
  - Prominent depth badge in Sensor Overview section

## Data Integration

### API Configuration
- **Centralized Config** - `src/lib/api-config.ts` for easy API switching
- **Public API** - `https://sensors.bgs.ac.uk/FROST-Server/v1.1` (current)
- **Internal API** - Available for BGS internal use

### Sensor Data
- **4 Major Sites** - UKGEOS Glasgow, BGS Cardiff, UKGEOS Cheshire, Wallingford
- **200+ Sensors** - Groundwater, weather, soil gas monitoring
- **Gas Measurements** - CO₂, CH₄, O₂ from GasClam probes
- **Environmental Data** - Temperature, pressure, humidity, conductivity
- **Real-time Updates** - Direct FROST API integration

## Performance & Optimization

### Latest Optimizations (August 2025)
- **Bundle Size** - 30% reduction: 19.1kB main page, 26.3kB sensor page
- **Code Cleanup** - 300+ lines of dead code removed
- **Centralized Functions** - API calls consolidated in `bgs-api.ts`
- **Memoized Dependencies** - Prevents unnecessary re-renders
- **TypeScript Compliance** - Zero compilation errors

### Data Limits
- **Default View** - 500 most recent readings (no date constraints)
- **Custom Ranges** - Up to 1000 readings for selected date periods
- **Optimized Loading** - Fast initial load, detailed exploration available
- **No Artificial Limits** - Fetches all available sensors and datastreams

## Styling & UI

- **shadcn/ui Components** - Always use first before custom components
- **Stone Theme** - Consistent with BGS branding
- **Dark Mode** - System-aware with localStorage persistence
- **UK Localisation** - "Normalised" spelling, en-GB formatting
- **Responsive Design** - Works on mobile and desktop

## Development Guidelines

### Code Style
- **Minimal & Clean** - No over-engineering
- **Single Responsibility** - Components focus on one task
- **Centralized Logic** - Common functions in `/lib`
- **Type Safety** - Strict TypeScript configuration
- **Performance First** - Memoization and optimized dependencies

### API Usage

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pautva) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
