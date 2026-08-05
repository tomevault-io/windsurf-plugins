---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Project Overview

Lepton is a lean code snippet manager powered by GitHub Gist, built with Electron, React, and Redux. It provides a desktop application for managing and organizing code snippets with features like unlimited public/secret snippets, tagging, markdown/Jupyter notebook support, and GitHub Enterprise integration.

## Tech Stack

- **Framework**: Electron (desktop app)
- **Frontend**: React 19 + Redux (with Redux Thunk for async actions)
- **Build System**: Webpack + Babel (ES6 transpilation)
- **Styling**: Sass/SCSS
- **Code Editor**: CodeMirror (via react-codemirror)
- **Testing/Linting**: Vitest, Electron smoke tests, ESLint with Standard config
- **Dependencies**: Uses npm package manager

## Key Commands

### Runtime

Use Node.js 24 LTS for local development, preferably the version in `.nvmrc`:
```bash
nvm install
nvm use
```

The project uses npm 11.x. This is the host runtime for dependency installation
and webpack builds; Electron's bundled app runtime is controlled separately by
the Electron version in `package.json`.

On Apple Silicon, use the native arm64 Node.js runtime:
```bash
npm ci
npm run build
npm start
```

### Development
```bash
# Install dependencies
npm ci

# Development build and run
npm run build && npm start

# Watch mode for development
npm run webpack-watch

# Production build
npm run webpack-prod
```

### Building & Distribution
```bash
# Create installer for current platform
npm run dist

# Platform-specific builds
npm run dist -- -m    # macOS
npm run dist -- -w    # Windows
npm run dist -- -l    # Linux (requires Docker for snap)
npm run dist -- -wml  # All platforms
```

### Code Quality
```bash
# Lint code
npm run lint

# Normal local check: Vitest unit tests plus webpack development build
npm test

# Unit tests only
npm run test:unit

# Unit tests in watch mode
npm run test:unit:watch

# Webpack build verification only
npm run test:build

# Check for outdated dependencies
npm run check-outdated

# Pre-version checks (runs lint + test + outdated check)
npm run preversion
```

### Testing
The `npm test` command runs Vitest unit tests and webpack development build
verification. Use it for normal local validation before committing code changes.

Electron smoke checks are available for UI/runtime-sensitive changes:

```bash
# Renderer smoke: login screens plus fixture-backed authenticated surfaces
npm run test:smoke

# Packaged app smoke: builds an unpacked app and verifies packaged login render
npm run test:packaged-smoke
```

`npm run test:smoke` uses isolated config/user-data directories. It verifies the
login UI and render fixtures such as active layout, edit/new modals,
about/settings, dashboard, search, delete confirmation, raw modal, pinned tags,
and immersive view. Fixture smoke verifies initial rendering, preload bridge
availability, renderer isolation, visible dimensions, screenshots, and absence
of renderer warnings/errors/crashes.

Fixture smoke does not validate GitHub OAuth, Gist CRUD, sync behavior, real API
responses, OS shortcut delivery, or full interaction flows. Use manual
verification for those.

### Rendering Verification
For any Electron, React, layout, CSS, webpack, or dependency change that could affect the UI, always verify actual rendering before considering the work complete:

```bash
npm run build
npm start
```

Confirm more than process startup. The app must visibly render the Lepton UI, not just log `updateUserSession ACTIVE`. Always capture a screenshot of the running app and show that screenshot in the chat when reporting rendering verification. If the window is blank or suspicious, inspect the renderer with Electron DevTools or a remote debugging port and verify DOM layout, visible text, and the renderer screenshot. Before relaunching, check for and stop duplicate Lepton/Electron instances so stale blank windows do not mask the current result.

For render-only confidence on authenticated surfaces, use the fixture smoke
path when appropriate:

```bash
npm run build
LEPTON_RENDER_FIXTURE=active npm start
```

Supported fixture names are `active`, `edit`, `new`, `about`, `dashboard`,
`search`, `delete`, `raw`, `pinned-tags`, and `immersive`. Fixture rendering is
acceptable for checking React/Electron render regressions without login. It is
not enough for changes touching GitHub authentication, backend sync, request
construction, shortcut delivery, or save/delete flows; those require manual
logged-in verification against GitHub/Gist behavior.

Document rendering verification in PR descriptions, including whether the app was launched locally and what was observed.

## Architecture

### Application Structure
- `/app` - Main React application source
  - `/containers` - React container components (connected to Redux)
  - `/reducers` - Redux reducers for state management
  - `/actions` - Redux action creators
  - `/utilities` - Shared utilities (GitHub API, parser, search, etc.)
- `/configs` - Configuration files including GitHub OAuth credentials
- `/main.js` - Electron main process entry point
- `/bundle` - Webpack build output directory

### Key Components

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hackjutsu/Lepton](https://github.com/hackjutsu/Lepton) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
