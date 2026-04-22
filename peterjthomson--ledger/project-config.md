---
trigger: always_on
description: This file helps AI assistants understand and work with the Ledger codebase.
---

# AGENTS.md

This file helps AI assistants understand and work with the Ledger codebase.

## Project Overview

Ledger is a macOS desktop app for viewing git branches, worktrees, and pull requests. Built with Electron + React + TypeScript.

## Quick Facts

| Aspect | Details |
|--------|---------|
| Type | Electron desktop app |
| Platform | macOS (Apple Silicon) |
| Language | TypeScript (strict mode) |
| UI | React 19 + custom CSS |
| Git | `simple-git` library |
| PRs | GitHub CLI (`gh`) |
| Tests | Playwright E2E |
| Build | electron-vite + electron-builder |

## Key Files to Know

```
lib/main/main.ts         # IPC handlers, app lifecycle
lib/services/            # Git operations as pure functions (branch, commit, stash, etc.)
lib/services/codegraph/  # Code dependency graph parsing (PHP, Ruby, TypeScript)
lib/conveyor/            # Typed IPC with Zod validation (schemas, handlers, APIs)
lib/preload/preload.ts   # API exposed to renderer
app/app.tsx              # Main React component (large)
app/styles/app.css       # All styling (large)
app/types/electron.d.ts  # TypeScript types for IPC
app/components/          # UI components (panels, canvas, window)
resources/scripts/       # External parser scripts (PHP, Ruby)
```

## Common Tasks

### Adding a new git operation

1. Add pure function to appropriate service in `lib/services/` (e.g., `branch-service.ts`)
2. Add Zod schema in `lib/conveyor/schemas/`
3. Add handler in `lib/conveyor/handlers/`
4. Add API method in `lib/conveyor/api/`
5. Call via `window.conveyor.*` from renderer

### Adding UI elements

Main UI is in `app/app.tsx`. Components are in `app/components/`:
- `panels/editor/` - Editor panels (BranchDetail, PRReview, Staging, etc.)
- `panels/viz/` - Visualization panels (GitGraph, ERD, CodeGraph)
- `panels/viz/codegraph/` - Code dependency graph visualization
- `canvas/` - Canvas layout system
- `window/` - Window chrome (Titlebar, menus)

Styling in `app/styles/app.css` uses CSS variables for theming.

### Running the app

```bash
npm run dev      # Development with hot reload
npm test         # Run E2E tests
npm run lint     # Check for linting issues
npm run build:mac:arm64  # Build for Apple Silicon
```

## Architecture Summary

```
Main Process (Node.js)
├── main.ts - App lifecycle, handler registration
├── lib/services/ - Pure functions for git operations
├── lib/conveyor/handlers/ - IPC handlers with Zod validation
└── settings-service.ts - Persistent storage

    ↕ IPC (conveyor: typed + validated)

Preload Script
└── preload.ts - Exposes window.electronAPI + window.conveyor

    ↕ contextBridge

Renderer Process (Browser)
└── app.tsx - React UI, state management
```

## State Management

Uses React hooks only (no Redux/Zustand):
- `useState` for data (branches, worktrees, prs, loading states)
- `useMemo` for derived data (filtered/sorted branches)
- `useCallback` for handlers
- `useEffect` for side effects

## Styling Approach

- Custom CSS (not Tailwind, despite it being installed)
- CSS variables for colors (`--accent`, `--bg-primary`, etc.)
- Mac native light theme aesthetic
- Responsive multi-column layout

## Testing

Playwright E2E tests in `tests/app.spec.ts`:
- Tests welcome screen (no repo)
- Tests main view (with repo via `--repo=` CLI arg)

Run with `npm test` (builds first) or `npm run test:headed`.

## Chrome DevTools Protocol (CDP) Access

AI agents can interact with the running Electron app via CDP for debugging and UI interaction.

```bash
# Start with CDP enabled
npm run dev -- --remote-debugging-port=9222

# Verify available
curl -s http://127.0.0.1:9222/json
```

Helper scripts in `tools/electron-mcp-server/`:

| Script | Purpose |
|--------|---------|
| `cdp-snapshot.js` | Get page content and text |
| `cdp-screenshot.js` | Capture PNG screenshot |
| `cdp-click.js` | Click element by CSS selector |

Usage: `cd tools/electron-mcp-server && node cdp-screenshot.js`

## Git Operations Available

The canonical sources are:

- `app/types/electron.d.ts` - Renderer-facing API types
- `lib/services/` - Git operations as pure functions
- `lib/conveyor/schemas/` - IPC channel definitions with Zod validation

## Code Graph (AST Parsing)

Visualizes code dependencies as a force-directed network graph. Supports:

| Language | Parser | Notes |
|----------|--------|-------|
| PHP | `nikic/php-parser` (bundled) | Laravel-optimized (Models/Controllers/Services) |
| Ruby | Regex-based | Rails-optimized (Models/Controllers) |
| TypeScript | `ts-morph` | Full AST with tsconfig resolution |

Key files:
- `lib/services/codegraph/` - Parser service and type definitions
- `resources/scripts/php-ast-parser.php` - PHP parser (bundled with nikic/php-parser)
- `resources/scripts/ruby-ast-parser.rb` - Ruby parser (no gem dependencies)
- `app/components/panels/viz/codegraph/` - React components and D3 renderer

The parsers run zero-touch (no modifications needed to target repo). For Laravel/Rails projects, only key directories are scanned for performance. Nodes with fewer than 2 connections are filtered out.

## Error Handling

- Git errors shown in error banner
- PR errors shown in PR column
- Operation results shown as dismissible toasts
- All IPC returns `{ success, message }` or `{ error }` pattern

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [peterjthomson/ledger](https://github.com/peterjthomson/ledger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
