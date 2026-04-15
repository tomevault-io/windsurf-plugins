---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Measured Munch** is a client-side Progressive Web App (PWA) for tracking intermittent fasting. It has no backend—all data persists in the browser's LocalStorage. The app deploys to a subdirectory path (`/measuredmunch/`) on applicaudia.se.

## Development Commands

```bash
# Install dependencies
bun install

# Start development server (runs on localhost with Vite)
bun run dev

# Run tests (Vitest with jsdom environment)
bun run test

# Lint code
bun run lint

# Format code with Prettier
bun run format

# Check code formatting (without modifying)
bun run format:check

# Build for production (TypeScript compilation + Vite build)
bun run build

# Preview production build locally
bun run preview

# Generate self-signed SSL cert for local HTTPS testing
bun run genkeys

# Serve production build with HTTPS (requires cert from genkeys)
bun run serve
```

## Architecture

### State Management Pattern

The app uses a **React Context + LocalStorage** pattern for all state:

- **`context.tsx`**: `LSProvider` wraps the app and provides centralized state management
  - Handles cross-tab synchronization via `storage` events
  - Manages initial sync from localStorage on app load
  - Prevents duplicate history records when syncing

- **`hooks.ts`**: Custom hooks that bridge React state with localStorage
  - `useLocalStorage<T>()`: Generic hook for any localStorage key
  - `useLSFastingHours()`: User's fasting duration setting
  - `useLSCurrentFastingState()`: Active fast state (startTime, isActive)
  - `useLSHistory()`: Fasting records with stats tracking
  - `useLSClearAll()`: Wipes all app data

- **`types.ts`**: Core TypeScript interfaces
  - `FastState`: `{ isActive: boolean, startTime: number | null }`
  - `FastSettings`: `{ fastingHours: number }`
  - `FastingRecord`: `{ startTime, endTime, durationMs, targetMs }`
  - `TimeRemaining`: Countdown timer state

### Key Data Flows

**Starting a fast:**
1. User clicks "Start Fast" → `setFastingState({ isActive: true, startTime: Date.now() })`
2. `useCountdown` hook begins 1-second interval updates
3. Timer calculates remaining time based on `startTime + fastingHours`

**Stopping a fast (early):**
1. User clicks "I'm Hungry" → creates `FastingRecord` with current elapsed time
2. `addHistoryRecord()` saves to localStorage (checks for duplicates first)
3. Fast state cleared

**Fast completion:**
1. `useCountdown` detects `isComplete`
2. `App.tsx` useEffect calls `stopFast()` automatically
3. Record added to history as "successful" if `durationMs >= targetMs`

### Cross-Tab Synchronization

The `LSProvider` listens for browser `storage` events to keep state in sync across multiple tabs:

- Storage events only fire in other tabs (not the originating tab)
- History syncing requires special handling: records must be re-added individually to recalculate stats correctly
- Initial sync runs on mount to populate context from localStorage

### Vite Configuration

- **Base path**: `/measuredmunch/` — required for subdirectory deployment
- **PWA**: `vite-plugin-pwa` with auto-update service worker
- **Manifest**: Defines icons, display mode (standalone), and theme color
- **Testing**: Vitest with jsdom, globals enabled, setup file at `src/setupTests.ts`

### Deployment

- **CI/CD**: GitHub Actions (`.github/workflows/cd.yml`)
  - Triggers: push to `main` or `devel`, all PRs
  - Runs: lint → type check → format check → tests → build → SFTP upload
  - Only deploys on push to `main`/`devel` (not PRs)
- **SFTP**: `scripts/batchfile_sftp.txt` uploads `dist/*` to `measuredmunch/` directory
- **Server**: Apache at applicaudia.se, no special routing needed (PWA handles its own routes)

## Important Constants

From `hooks.ts`:
- `DEFAULT_FASTING_HOURS = 16`
- `MIN_FASTING_MS = 5000` — minimum duration to count as a valid fast (prevents accidental clicks)
- LocalStorage keys: `measuredMunch:settings`, `measuredMunch:state`, `measuredMunch:history`

## AI Instructions Sync

**IMPORTANT**: `CLAUDE.md` is the **Single Source of Truth (SSOT)** for AI agent instructions. Any changes to `CLAUDE.md` should be followed by running the copy script to propagate changes to other AI tools.

The `scripts/copy-ai-instructions.sh` script keeps AI assistant instructions in sync:
- Source: `CLAUDE.md` (SSOT)
- Copies to: `GEMINI.md` and `.github/copilot-instructions.md`
- **Run this after updating CLAUDE.md** to propagate changes to other AI tools

```bash
./scripts/copy-ai-instructions.sh
```

## Design System

The comprehensive design system is documented in `docs/STYLE_GUIDE.md`. This guide covers:
- **Color Palette**: Electric Blue accent with Zinc/Slate dark theme
- **Typography System**: Inter font family with weight scale
- **Spacing Scale**: 4px base unit system
- **Component Patterns**: Bento grid variants, buttons, progress bars
- **Animation Guidelines**: Duration and easing functions
- **Accessibility Standards**: WCAG AA compliance, focus states, reduced motion
- **Responsive Breakpoints**: Mobile, tablet, desktop layouts

When making UI changes, reference the style guide to maintain consistency.

## Testing


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cfogelklou) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
