---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CCSeva is a macOS menu bar Electron application that monitors Claude Code usage in real-time. The app uses the `ccusage` npm package API to fetch token usage data and displays it through a modern React-based UI with tabbed navigation, analytics, notifications, and visualizations.

## Essential Commands

### Development
```bash
npm run electron-dev  # Start with hot reload (recommended for development)
npm run dev           # Build frontend only in watch mode
npm start            # Start built app
```

### Building
```bash
npm run build        # Production build (webpack + tsc compilation)
npm run pack         # Package app with electron-builder
npm run dist         # Build and create distribution package
npm run dist:mac     # Build for macOS specifically
```

### Code Quality
```bash
npm run lint         # Run Biome linter
npm run lint:fix     # Fix linting issues automatically
npm run format       # Format code with Biome
npm run format:check # Check code formatting
npm run check        # Run linting and formatting checks
npm run check:fix    # Fix linting and formatting issues
npm run type-check   # TypeScript type checking without emit
```

### Dependencies
```bash
npm install          # Install all dependencies
```

## Architecture Overview

### Dual-Process Electron Architecture
The app follows standard Electron patterns with clear separation:

- **Main Process** (`main.ts`): Manages system tray, IPC, and background services
- **Renderer Process** (`src/`): React app handling UI and user interactions
- **Preload Script** (`preload.ts`): Secure bridge exposing `electronAPI` to renderer

### Key Architectural Components

#### Service Layer (Singleton Pattern)
- **CCUsageService**: Uses the `ccusage` npm package data-loader API to fetch usage data, implementing a 30-second cache. Now supports plan configuration and actual session-based reset times.
- **SettingsService**: Manages user preferences persistence to `~/.ccseva/settings.json` including plan selection, custom token limits, timezone, and reset hour settings
- **NotificationService**: Manages macOS notifications with cooldown periods and threshold detection
- **ResetTimeService**: Handles Claude usage reset time calculations and timezone management
- **SessionTracker**: Tracks user sessions and activity patterns for analytics

#### Data Flow
1. Main process polls CCUsageService every 30 seconds
2. Service imports `loadSessionBlockData` and `loadDailyUsageData` from `ccusage/data-loader` to fetch usage data
3. The returned JavaScript objects are mapped to typed interfaces (`UsageStats`, `MenuBarData`)
4. Menu bar updates with percentage display, renderer receives data via IPC
5. React app renders tabbed interface with dashboard, analytics, and live monitoring views
6. NotificationService triggers alerts based on usage thresholds and patterns

#### Modern UI Component Architecture
```
App.tsx (main container with state management)
├── NavigationTabs (tabbed interface)
├── Dashboard (overview with stats cards)
├── LiveMonitoring (real-time usage tracking)
├── Analytics (charts and historical data)
├── TerminalView (command-line interface simulation)
├── SettingsPanel (user preferences)
├── LoadingScreen (app initialization)
├── ErrorBoundary (error handling)
├── NotificationSystem (toast notifications)
└── ui/ (Radix UI components)
    ├── Button, Card, Progress, Tabs
    ├── Alert, Badge, Tooltip, Switch
    └── Avatar, Popover, Select, Slider
```

### Build System Specifics

#### Dual Compilation Process
The build requires both Webpack (renderer) and TypeScript compiler (main/preload):
```bash
webpack --mode production && tsc main.ts preload.ts --outDir dist
```

#### Critical Path Dependencies
- **ccusage npm package**: Direct dependency providing data-loader API functions
- **Tailwind CSS v3**: PostCSS processing with custom gradient themes
- **React 19**: Uses new JSX transform (`react-jsx`)
- **Radix UI**: Component library for accessible UI primitives
- **Biome**: Fast linter and formatter replacing ESLint/Prettier

### IPC Communication Pattern

Main process exposes these handlers:
- `get-usage-stats`: Returns full UsageStats object
- `refresh-data`: Forces cache refresh and returns fresh data
- `usage-updated`: Event emitted to renderer every 30 seconds

Renderer accesses via `window.electronAPI` (type-safe interface in preload.ts).

## Data Processing Logic

### Usage Calculation
The app detects Claude plans automatically:
- **Pro**: ≤7,000 tokens
- **Max5**: ≤35,000 tokens  
- **Max20**: ≤140,000 tokens
- **Custom**: >140,000 tokens

### Burn Rate Algorithm
Calculates tokens/hour based on last 24 hours of usage data, used for depletion time predictions.

### Error Handling Strategy
- CCUsageService returns default stats on ccusage command failures
- React components display error states with retry buttons
- Main process continues functioning even if data fetch fails

## Development Considerations

### TypeScript Configuration
Uses strict mode with custom path aliases (`@/*` → `src/*`). Three separate tsconfig files:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Iamshankhadeep/ccseva](https://github.com/Iamshankhadeep/ccseva) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
