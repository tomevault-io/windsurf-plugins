---
trigger: always_on
description: Project-wide coding conventions, build commands, and architecture patterns
---


# Timestamp Project Instructions

A customizable time tracking app with countdowns, timers, and world clocks featuring multiple visual themes and instant URL sharing. This file provides project-wide coding conventions, build commands, and architecture patterns for GitHub Copilot.

## Key Concepts

### Project Overview

TypeScript project featuring:
- **Three countdown modes**:
  - **🏠 Local Time** (Wall clock): Per timezone, e.g. New Year's Eve — each timezone celebrates independently
  - **🌐 Same Moment** (Absolute time): One instant, e.g. product launch — everyone counts to the same UTC instant
  - **⏱️ Timer** (Your countdown): Fixed duration countdown — count down from any duration
- **Instant URL sharing**: Every countdown configuration generates a shareable URL
- **Multiple visual themes**: GitHub contribution graph aesthetic, fireworks, and more

### Project Structure

```
timestamp/
├── src/
│   ├── app/                    # Orchestrator & state coordination
│   │   ├── orchestrator/       # Modular orchestrator components
│   │   │   ├── controllers/    # Page lifecycle controllers
│   │   │   ├── theme-manager/  # Theme loading & transitions
│   │   │   ├── time-manager/   # Countdown loop & timer controls
│   │   │   └── ui/             # UI chrome visibility & colors
│   │   └── pwa/                # PWA registration & updates
│   ├── components/             # UI components
│   │   ├── color-mode-toggle/  # Light/dark mode toggle
│   │   ├── countdown-buttons/  # Timer controls (play/pause/reset)
│   │   ├── landing-page/       # Landing page modules
│   │   ├── mobile-menu/        # Mobile hamburger menu
│   │   ├── perf-overlay/       # Performance monitoring overlay
│   │   ├── pwa/                # PWA UI components
│   │   ├── theme-picker/       # Theme selection modal
│   │   ├── timezone-selector/  # Timezone selector
│   │   ├── toast/              # Toast notification system
│   │   ├── tooltip/            # Tooltip component
│   │   └── world-map/          # Day/night visualization
│   ├── core/                   # Core types, state, utilities
│   │   ├── config/             # Mode configuration
│   │   ├── state/              # App state management
│   │   ├── types/              # Shared type definitions
│   │   ├── time/               # Time calculations & timezones
│   │   ├── url/                # URL building & parsing
│   │   └── utils/              # Accessibility, DOM, performance
│   ├── data/                   # Static data (cities, SVG paths)
│   ├── styles/                 # CSS files
│   ├── test-utils/             # Shared test helpers
│   └── themes/                 # Theme implementations
│       ├── registry/           # SINGLE SOURCE OF TRUTH
        ├── theme-1/
        ├── theme-2/
        ......
│       └── shared/             # Shared cleanup, constants
├── docs/                       # Specs, plans, guides
├── e2e/                        # Cross-cutting E2E tests
├── scripts/                    # Build scripts
└── .github/                    # Workflows, agents, instructions, prompts
```

---

## Rules and Guidelines

### Key Features
- **Local Time** (wall-clock): Count down to specific dates per timezone (New Year's Eve, birthdays)
- **Same Moment** (absolute): Count down to a single instant globally (product launches, events)
- **Timer**: Fixed duration countdowns (5 minutes, 2 hours, custom durations)
- **URL Sharing**: Automatically generates shareable URLs for any countdown configuration
- **Multiple Themes**: 
  - Contribution Graph: GitHub-style grid with pulsing digits and living background
  - Fireworks: Dynamic canvas animations that intensify as time approaches zero
- **World Map**: Day/night visualization showing real-time solar position
- **Timezone Support**: Full IANA timezone database with intelligent defaults
- **Accessibility**: Screen reader support and reduced motion
- **Responsive Layout System**:
  - Safe Area: Themes render within safe area defined by CSS custom properties (`--safe-area-*`)
  - Breakpoints: mobile (≤1050px shows hamburger), desktop (>1050px)
  - Mobile Hamburger Menu: All chrome consolidated in overlay on mobile for maximum countdown space
  - Responsive Fonts: `--font-scale` CSS property (0.7 mobile, 1.0 desktop)
  - Utility module: `@themes/shared/responsive-layout`

### Development Commands
- `npm install` - Install dependencies
- `npm run dev` - Start development server (default port: 5173)
- `npm run build` - Build for production
- `npm run test` - Run unit tests with Vitest
- `npm run test:e2e` - **Default**: Fast E2E tests (chromium only, excludes long-running @perf tests)
- `npm run test:e2e:cross-browser` - Cross-browser E2E tests (chromium, webkit, mobile)
- `npm run test:e2e:perf` - All performance tests (quick + deep)
- `npm run test:e2e:perf:quick` - Quick perf tests (10s each, parallel workers)
- `npm run test:e2e:perf:deep` - Deep profiling tests (60s+ each, sequential)
- `npm run test:e2e:perf:all` - Performance profiling for ALL themes (audit mode)
- `PERF_THEME=<id> npm run test:e2e:perf` - Performance profiling for a SPECIFIC theme
- `npm run test:e2e:full` - Complete E2E suite including performance tests (for CI)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chrisreddington/timestamp](https://github.com/chrisreddington/timestamp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
