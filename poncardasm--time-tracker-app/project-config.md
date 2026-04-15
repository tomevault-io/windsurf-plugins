---
trigger: always_on
description: Generates a Svelte Playground link with the provided code.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a minimalist Progressive Web App (PWA) for time tracking built with Svelte 5. It's a client-side-only application with no backend - all data is stored in browser localStorage. The app supports both stopwatch and Pomodoro timer modes, with real-time tracking, manual time entry, project categorization, dark mode support, and CSV export functionality.

## Tech Stack

- **Build Tool**: Vite 6.x (latest stable)
- **Frontend Framework**: Svelte 5.0.0
- **Styling**: Tailwind CSS 4.1.0 (PostCSS processed)
- **PWA**: Service Worker with network-first caching strategy
- **Data Persistence**: Browser localStorage

## Architecture

### Core Application Flow

1. **State Management**: The app uses Svelte 5 runes ($state, $derived, $effect, $props) for reactive state management, organized into three store modules:

   - `stores/tasks.svelte.js` - Task data and CRUD operations (src/stores/tasks.svelte.js)
   - `stores/timer.svelte.js` - Active task timer, elapsed time, and Pomodoro state (src/stores/timer.svelte.js)
   - `stores/theme.svelte.js` - Dark mode state and persistence (src/stores/theme.svelte.js)

2. **Data Storage** (src/stores/tasks.svelte.js:1-33): All task records are stored in localStorage under the key 'timeTrackerTasks' as JSON. The tasks store exports functions for CRUD operations.

3. **Timer System** (src/stores/timer.svelte.js:24-62): Uses `setInterval` for live tracking with two modes:
   - **Stopwatch mode**: Calculates elapsed time by comparing current time with `startTime`
   - **Pomodoro mode**: Counts down from 25 minutes (work) and 5 minutes (break), with automatic phase switching and browser notifications

### Key Components

- **App.svelte** (src/App.svelte): Root component managing application-level state including modal visibility, modal modes, and event handlers
- **Header.svelte**: App header with dark mode toggle button
- **StartView.svelte**: Initial view with "Start Task" and "Add Manual Entry" buttons
- **ActiveView.svelte**: Active timer display showing elapsed time (stopwatch) or countdown (Pomodoro), with stop button
- **TaskModal.svelte** (src/components/TaskModal.svelte): Tri-modal design supporting 'start', 'manual', and 'edit' modes with project autocomplete dropdown
- **DeleteModal.svelte**: Confirmation modal for bulk task deletion
- **HistoryList.svelte**: Task history table with inline edit buttons, checkbox selection, and CSV export
- **Theme System** (src/stores/theme.svelte.js): Dark mode with localStorage persistence and system preference detection

### Component Structure

```
src/
├── App.svelte              # Root component, manages modals and event handlers
├── main.js                 # Entry point, mounts Svelte app
├── main.css                # Tailwind CSS imports and custom styles
├── utils.js                # Utility functions (formatTime, toLocalISOString)
├── stores/
│   ├── tasks.svelte.js     # Task data store with CRUD operations
│   ├── timer.svelte.js     # Timer state and Pomodoro logic
│   └── theme.svelte.js     # Dark mode state management
└── components/
    ├── Header.svelte       # Header with dark mode toggle
    ├── StartView.svelte    # Start/manual entry buttons
    ├── ActiveView.svelte   # Active timer display
    ├── TaskModal.svelte    # Task creation/editing modal
    ├── DeleteModal.svelte  # Delete confirmation modal
    └── HistoryList.svelte  # Task history table and export
```

### Data Structure

Task records follow this schema:

```javascript
{
    taskName: string,      // User-provided description
    project: string,       // Optional project category (nullable)
    startTime: number,     // Unix timestamp (ms)
    endTime: number,       // Unix timestamp (ms)
    durationMs: number     // Calculated duration in milliseconds
}
```

Active task state (during tracking):

```javascript
{
    description: string,   // Task description
    project: string,       // Optional project category
    startTime: number,     // Unix timestamp when started
    mode: string          // 'stopwatch' or 'pomodoro'
}
```

## Development Workflow

### Installing Dependencies

First, install the required dependencies:

```bash
npm install
```

### Running the Application

#### Development Mode

Start the Vite development server with hot module replacement:

```bash
npm run dev
```

The app will automatically open at <http://localhost:8000>. Changes to source files will be reflected immediately thanks to Svelte's HMR.

#### Production Build

Build the optimized production bundle:

```bash
npm run build
```

Output will be in the `dist/` directory.

#### Preview Production Build

Preview the production build locally:

```bash
npm run preview
```

### Testing PWA Functionality

1. Serve over HTTPS (required for service workers) or use localhost
2. Open DevTools → Application → Service Workers to monitor SW lifecycle
3. Test offline: DevTools → Network → Set to "Offline" mode

### Service Worker Cache Updates

When modifying source files or other cached resources:

1. Increment the `CACHE_NAME` version in public/sw.js (currently 'time-tracker-v4')

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/poncardasm) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
