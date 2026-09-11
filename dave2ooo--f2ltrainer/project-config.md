---
trigger: always_on
description: F2LTrainer is a web-based training application for learning F2L (First Two Layers) algorithms for Rubik's cube solving. The application helps users practice and master various F2L cases through interactive training sessions.
---

# GitHub Copilot Instructions for F2LTrainer

## Project Overview

F2LTrainer is a web-based training application for learning F2L (First Two Layers) algorithms for Rubik's cube solving. The application helps users practice and master various F2L cases through interactive training sessions.

## Critical Rules

### NO Unnecessary Refactoring

**IMPORTANT**: When making changes to implement a feature or fix a bug:

- **ONLY modify files directly related to the functional change**
- **DO NOT reformat or refactor unrelated files**
- **DO NOT apply code style changes to files that don't need functional modifications**
- **DO NOT run formatters or linters on the entire codebase** - only on files you're actively changing
- If a file is working and doesn't need functional changes, **leave it untouched**

Examples of what NOT to do:

- ❌ Reformatting algorithm or scramble data files when adding a UI feature
- ❌ Changing whitespace or code style in files you're not functionally modifying
- ❌ Refactoring utility functions when fixing an unrelated bug
- ❌ Updating type definitions that don't need changes for your feature

Examples of acceptable changes:

- ✅ Adding a new prop to a component that needs the feature
- ✅ Creating new utility functions for new functionality
- ✅ Updating types when the feature requires new or modified types
- ✅ Formatting **only** the lines you are adding or modifying

### Keep Changes Minimal and Focused

- Make the smallest possible change to achieve the goal
- If adding a feature requires changes to 3 files, your PR should touch exactly 3 files (plus tests)
- Before committing, review **every file** that was modified and ask: "Did this file need a functional change?"
- If the answer is no, revert that file

## Technology Stack

- **Framework**: SvelteKit with Svelte 5 (using Svelte 5 runes for reactivity)
- **Language**: TypeScript (strict mode enabled)
- **Styling**: TailwindCSS v4 with Flowbite components
- **UI Components**: Flowbite-Svelte, Flowbite-Svelte-Icons, Flowbite-Svelte-Blocks
- **Build Tool**: Vite v7
- **Package Manager**: npm (with pnpm support via lockfile)
- **3D Cube Visualization**: cubing library
- **Documentation**: MDSvex for markdown support

## Code Style & Formatting

### Prettier Configuration

- **Indentation**: Tabs (not spaces)
- **Quotes**: Single quotes
- **Trailing Commas**: None
- **Print Width**: 100 characters
- **Plugins**: prettier-plugin-svelte, prettier-plugin-tailwindcss

### ESLint Rules

- TypeScript recommended rules enabled
- Svelte recommended rules enabled
- `no-undef` rule disabled for TypeScript files (TypeScript handles this)
- Browser and Node globals available

### Code Formatting Commands

- Format code: `npm run format`
- Check formatting: `npm run lint`
- Type checking: `npm run check`

**IMPORTANT**: DO NOT run `npm run format` on the entire codebase. Only format files you are actively modifying. Use your editor's format-on-save or manually format specific files to avoid reformatting unrelated code.

## Project Structure

```
src/
├── lib/                      # Library code and components
│   ├── components/          # Svelte components
│   │   ├── SelectView/     # Case selection view
│   │   ├── TrainView/      # Training view
│   │   └── Modals/         # Modal dialogs
│   ├── data/               # Static data and configurations
│   ├── types/              # TypeScript type definitions
│   ├── utils/              # Utility functions
│   ├── casesState.svelte.ts    # Case state management
│   ├── globalState.svelte.ts   # Global application state
│   └── trainCaseQueue.svelte.ts # Training queue state
├── routes/                  # SvelteKit routes
│   ├── +layout.svelte      # Root layout
│   ├── +layout.ts          # Layout load function
│   └── +page.svelte        # Main page
└── app.css                 # Global styles

e2e/                        # Playwright end-to-end tests
```

## State Management

This project uses **Svelte 5 runes** for state management:

- `$state()` for reactive state
- `$derived()` for computed values
- `$effect()` for side effects
- `.svelte.ts` files for shared state modules

**Key State Files:**

- `globalState.svelte.ts`: Application-wide settings and UI state
- `casesState.svelte.ts`: F2L case states (unlearned, learning, finished)
- `trainCaseQueue.svelte.ts`: Training session queue and progress

State is persisted to localStorage using utility functions in `utils/localStorage.ts`.

## Development Workflow

### Setup

```bash
npm install          # Install dependencies (requires Node.js >=22.3.0)
npm run dev          # Start development server (http://localhost:5173)
```

### Build & Deploy

```bash
npm run build        # Production build
npm run preview      # Preview production build
npm run pages:prepare # Prepare for GitHub Pages deployment
```

### Testing

- **Unit Tests**: Vitest with browser environment for Svelte component tests
  - Client tests: `src/**/*.svelte.{test,spec}.{js,ts}`
  - Server tests: `src/**/*.{test,spec}.{js,ts}` (excluding .svelte tests)
  - Run: `npm run test:unit`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dave2ooo/F2LTrainer](https://github.com/Dave2ooo/F2LTrainer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
