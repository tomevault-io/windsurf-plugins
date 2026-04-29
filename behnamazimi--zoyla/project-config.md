---
trigger: always_on
description: **Zoyla** is a cross-platform desktop load testing application built with:
---

# Zoyla - Agent Context Guide

## Project Overview

**Zoyla** is a cross-platform desktop load testing application built with:

- **Frontend**: React + TypeScript + Vite + Zustand + vanilla-extract
- **Backend**: Rust (via Tauri v2)

### Core Features

- **Load Testing**: Configurable HTTP load tests with real-time progress
- **Results Visualization**: Charts, histograms, percentiles, error logs
- **Test History**: Persistent storage of previous test runs
- **Export**: JSON and CSV export of test results
- **Advanced Options**: HTTP/2, rate limiting, proxy support, header randomization
- **Keyboard Shortcuts**: Full keyboard navigation support
- **Theme Support**: Dark/light theme with persistence

---

## Architecture

### Directory Structure

```
src/
├── styles/              # Global styles & theme tokens
│   ├── global.css.ts    # Global reset & base styles
│   ├── theme.css.ts     # Theme tokens (colors, spacing, etc.)
│   └── index.ts
├── components/          # Presentational (dumb) components - NO store imports
│   ├── buttons/         # Button, IconButton, CopyButton, Tooltip
│   ├── charts/          # Chart components (Area, Bar, Line, Scatter, ChartTooltip)
│   ├── feedback/        # ErrorMessage, ProgressBar, StatCard, StatusBadge
│   ├── forms/           # NumberInput, Select, TextInput, ToggleGroup
│   └── layout/          # Panel, ResizeHandle
├── features/            # Container (smart) components - connect to stores
│   ├── history/         # HistoryPanel, HistoryEntry
│   ├── results/         # ResultsContainer, SummaryPanel, ChartsGrid, StatusCodesPanel, ErrorLogsPanel
│   ├── settings/        # LayoutSettings
│   ├── shortcuts/       # KeyboardGuide
│   ├── test-config/     # TestConfigPanel, HeadersEditor
│   └── test-runner/     # RunButton, ProgressView
├── layouts/             # Page-level layout components
│   ├── AppShell.tsx     # Root layout wrapper
│   ├── Toolbar.tsx      # Top toolbar with history/settings buttons
│   ├── Sidebar.tsx      # Left sidebar (config + run button)
│   └── MainContent.tsx  # Right main area (progress/results)
├── store/               # Independent Zustand stores
│   ├── testConfigStore.ts    # Test configuration state
│   ├── testRunnerStore.ts    # Test execution state
│   ├── historyStore.ts       # Test history state
│   └── uiStore.ts            # UI preferences state
├── services/            # Pure async functions (no state)
│   ├── clipboard/       # Image clipboard operations
│   ├── export/          # CSV and JSON export
│   ├── storage/         # Persistent storage (history, settings, theme)
│   └── tauri/           # Tauri API calls (loadTest, events)
├── hooks/               # Custom React hooks
│   ├── useTestRunner.ts      # Orchestrates test execution (ONLY cross-store hook)
│   ├── useKeyboardShortcuts.ts # Global keyboard shortcuts
│   ├── usePersistence.ts      # Loads persisted data on mount
│   └── useResizable.ts        # Sidebar resize functionality
├── types/               # TypeScript type definitions
│   ├── api.ts           # Types matching Rust structs (TestConfig, LoadTestStats, etc.)
│   ├── store.ts         # Store state and action types
│   └── components.ts    # Component prop types
├── utils/               # Pure utility functions
│   ├── format.ts        # Number/date formatting
│   └── transform.ts     # Data transformations for charts
├── constants/           # Default values and limits
│   └── defaults.ts      # DEFAULT_TEST_CONFIG, MAX_REQUESTS, etc.
├── App.tsx              # Root component
└── main.tsx             # React entry point

src-tauri/
├── src/
│   ├── lib.rs           # Core Rust logic: load test execution, stats calculation
│   └── main.rs          # Tauri entry point
├── Cargo.toml           # Rust dependencies
└── tauri.conf.json      # Tauri configuration
```

### Key Architectural Rules

1. **Stores are INDEPENDENT** - Never import each other directly
2. **Cross-store coordination** happens ONLY in `hooks/useTestRunner.ts`
3. **Presentational components** (`components/`) have NO store imports
4. **Container components** (`features/`) connect to stores and pass props down
5. **Services are pure functions** - No state, just async operations
6. **Type safety** - `types/api.ts` mirrors Rust structs exactly

---

## Key Technologies

### Frontend Stack

| Package               | Purpose                                                    |
| --------------------- | ---------------------------------------------------------- |
| `react` + `react-dom` | UI framework                                               |
| `zustand`             | State management (lightweight, no providers)               |
| `vanilla-extract`     | CSS-in-TypeScript (zero-runtime, type-safe)                |
| `@tauri-apps/api`     | Tauri bridge for Rust communication                        |
| `recharts`            | Chart library for data visualization                       |
| `lucide-react`        | Icon library                                               |
| `@radix-ui/*`         | Accessible UI primitives (Popover, Dropdown, Select, etc.) |
| `html-to-image`       | Screenshot generation for export                           |

### Backend Stack (Rust)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [behnamazimi/zoyla](https://github.com/behnamazimi/zoyla) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
