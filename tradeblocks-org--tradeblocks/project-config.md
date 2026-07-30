---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TradeBlocks is a Next.js 15 application for analyzing options trading performance. It processes CSV exports of trade logs and daily portfolio logs to calculate comprehensive portfolio statistics, drawdowns, and performance metrics. The application uses IndexedDB for client-side storage of trading data.

## Development Commands

### Running the Application

- `npm run dev` - Start development server with Turbopack
- `npm run build` - Build production bundle with Turbopack
- `npm start` - Start production server

### Testing

- `npm test` - Run all tests with Jest
- `npm run test:watch` - Run tests in watch mode
- `npm run test:coverage` - Generate coverage report
- `npm run test:portfolio` - Run portfolio stats tests specifically

To run a single test file:

```bash
npm test -- path/to/test-file.test.ts
```

To run a specific test case:

```bash
npm test -- path/to/test-file.test.ts -t "test name pattern"
```

### Code Quality

- `npm run lint` - Run ESLint on the codebase

## Architecture

### Core Data Flow

1. **Data Import**: Users upload CSV files (trade logs and optional daily logs)
2. **Processing Pipeline**:
   - CSV parsing (`lib/processing/csv-parser.ts`)
   - Trade/daily log processing (`lib/processing/trade-processor.ts`, `lib/processing/daily-log-processor.ts`)
   - Data validation (`lib/models/validators.ts`)
3. **Storage**: Data stored in IndexedDB via store modules (`lib/db/`)
4. **Calculation**: Portfolio statistics calculated via `lib/calculations/portfolio-stats.ts`
5. **State Management**: Zustand stores (`lib/stores/`) manage UI state and coordinate data access

### Key Architectural Patterns

**Block-Based Organization**: Trading data is organized into "blocks" - each block represents a trading portfolio/strategy with:

- Trade log (required): Individual trade records
- Daily log (optional): Daily portfolio values for enhanced performance calculations
- Calculated statistics cached for performance

**Dual Storage Pattern**:

- Raw trade/daily log data → IndexedDB (via `lib/db/`)
- UI state & metadata → Zustand stores (via `lib/stores/`)
- This separation allows efficient data handling for large datasets

**Math.js for Statistical Calculations**: All statistics use `math.js` library to ensure consistency:

- Sharpe Ratio: Uses sample standard deviation (N-1) via `std(data, 'uncorrected')`
- Sortino Ratio: Uses standard downside deviation = sqrt((1/N) \* sum(min(excess_i, 0)^2)) where N = total observations. This is the RMS of negative excess returns from zero, NOT std() of only the negative returns.

### Directory Structure

- `app/` - Next.js 15 app router pages and layouts
  - `(platform)/` - Main application routes with sidebar layout
- `components/` - React components
  - `ui/` - shadcn/ui components (Radix UI primitives)
  - `performance-charts/` - Plotly-based performance visualizations (via react-plotly.js)
- `lib/` - Core business logic (framework-agnostic)
  - `models/` - TypeScript interfaces and types
  - `processing/` - CSV parsing and data processing
  - `calculations/` - Portfolio statistics calculations
  - `db/` - IndexedDB operations
  - `stores/` - Zustand state management
- `tests/` - Jest test suites
  - `unit/` - Unit tests for calculations and processing
  - `integration/` - Integration tests for data flow
  - `data/` - Mock data and test fixtures

### Critical Implementation Details

**Timezone Handling**: All dates and times are processed and displayed as **US Eastern Time** (America/New_York). This is critical because:

- Trading data originates from US markets operating on Eastern Time
- CSVs contain dates/times in Eastern Time format
- When parsing dates, preserve the calendar date as-is (don't convert to UTC)
- When displaying times, show Eastern Time (with DST awareness)
- Use `toLocaleDateString('en-US')` or manual string extraction instead of `.toISOString()` which converts to UTC
- Static datasets in `tests/data/` explicitly handle Eastern Time with DST awareness

**Date Comparison Rules (MCP Server)**: There are **two kinds of dates** in the MCP server, and they require different handling:

1. **Calendar dates from CSVs** (trade-log exports): These are Eastern Time trading dates like "2025-01-07" parsed via `parseDatePreservingCalendarDay()` → `new Date(year, month, day)`. The Date is created at **local midnight**, NOT Eastern midnight. The calendar date "7" is just temporarily stored inside a Date object — it's not a real timestamp. To read it back, you MUST use the same local-timezone methods (`getFullYear`/`getMonth`/`getDate`), which always return the original calendar date regardless of server timezone. This works because the write path (constructor) and read path (getters) both use local timezone — they're symmetric and cancel out.

2. **Absolute timestamps** (TradingView Unix epoch in market CSVs): These ARE real UTC instants representing a specific moment in time. To get the correct Eastern trading date, you MUST convert to ET via `toLocaleDateString("en-CA", { timeZone: "America/New_York" })`. This is the one place ET conversion is correct.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tradeblocks-org/tradeblocks](https://github.com/tradeblocks-org/tradeblocks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
