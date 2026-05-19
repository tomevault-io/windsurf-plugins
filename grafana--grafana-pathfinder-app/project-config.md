---
trigger: always_on
description: Describes technologies, frameworks, and other tools used in this repository.
---

# Tech Context

## Technologies Used

- **Frontend**: React 18.3.1 + TypeScript 5.9.3 + Grafana Scenes 7.0.3
- **Backend**: Go 1.23+ with grafana-plugin-sdk-go v0.290.0
- **Styling**: Emotion CSS-in-JS with Grafana UI theming system
- **State Management**: Grafana Scenes for complex scene-based state
- **Bundling**: Webpack 5.102.1 with custom configuration
- **Testing**: Jest 30.2.0 + React Testing Library + Playwright 1.56.1 for E2E, Go testing for backend
- **Runtime**: Node.js 22+ with npm 11.6.2 package management

## Development Setup

- **Build System**: Webpack with TypeScript, SWC compilation, and hot reloading (frontend); Mage for Go backend
- **Dev Environment**: Docker Compose with Grafana OSS for local testing
- **Scripts**: `npm run dev` (watch mode), `npm run build` (production), `npm run server` (Docker)
- **Go Build**: `mage build:darwin` (macOS), `mage build:linux` (Linux), `npm run build:backend` (Linux via npm)
- **Code Quality**: ESLint + Prettier with Grafana configs, TypeScript strict mode; `golangci-lint` for Go
- **Testing**: `npm run test:ci` (Jest CI mode), `npm run test:go` (Go tests), `npm run e2e` (Playwright), `npm run typecheck`

## Technical Constraints

- **Grafana Version**: Requires Grafana >=12.3.0-0 for extension points compatibility
- **Plugin Architecture**: Must use Grafana's app plugin structure with `plugin.json`
- **Extension Points**: Limited to `grafana/extension-sidebar/v0-alpha` integration
- **Browser Support**: Modern browsers only (ES2020+), no IE support
- **Bundle Size**: Webpack optimization required for performance in Grafana context

## Dependencies

**Frontend Runtime**:
- `@grafana/data`, `@grafana/ui`, `@grafana/runtime`, `@grafana/scenes` (12.4.0 / 7.0.3)
- `react` + `react-dom` (18.3.1), `react-router-dom` (6.28.0)
- `@emotion/css` (11.13.5) for styling
- `@dnd-kit/core`, `@dnd-kit/sortable`, `@dnd-kit/utilities` for drag-and-drop

**Backend Runtime** (Go):
- `grafana-plugin-sdk-go` (v0.290.0) - Grafana plugin SDK
- `gorilla/websocket` (v1.5.3) - WebSocket connections
- `golang.org/x/crypto` - SSH/crypto utilities

**Development**:
- `typescript` (5.9.3), `webpack` (5.102.1) + loaders, `jest` (30.2.0) + testing utilities
- `@grafana/eslint-config`, `@playwright/test` (1.56.1), `@swc/core` (1.15.1) for compilation
- `sass` (1.94.0), `terser-webpack-plugin` (5.3.14) for asset processing
- `mage` (v1.15.0) - Go build tool

## Project Version & Release Management

- **Current Version**: 1.1.71 (see package.json)
- **License**: Apache-2.0
- **Package Manager**: npm@11.6.2 with lockfile-based dependency management
- **Release Strategy**: Semantic versioning with automated plugin signing

## Tool Usage Patterns

- **TypeScript**: Strict mode with comprehensive type definitions for all components
- **Component Architecture**: Functional components with hooks, no class components
- **Styling**: Emotion CSS-in-JS with `useStyles2` hook and Grafana theme integration
- **Testing Strategy**: Unit tests with Jest, component tests with RTL, E2E with Playwright
- **Code Organization**: Engine-based modules with clear separation of concerns (interactive-engine, context-engine, requirements-manager)
- **Build Pipeline**: Development with watch mode, production with optimization and signing
- **Drag-and-Drop**: @dnd-kit library for all sortable/draggable interactions (see `src/components/block-editor/dnd/`)

## Current Architecture & Data Flow Overviews

Architecture: Layered system with Context System → Documentation Rendering → 
Interactive Guide System. For detailed architecture and data flows, see `docs/architecture.dot`.
Do NOT read `architecture.dot` unless you're working on cross-component changes or need to understand system-wide data flows.

## Key Architectural Patterns

### 1. Context Detection (Automatic & Continuous)
- **EchoSrv Integration**: Listens to Grafana analytics events for datasource/viz selection
- **Location Monitoring**: Tracks URL changes via LocationService and browser events
- **Debounced Updates**: Centralized timeout manager prevents rapid-fire API calls
- **Event Buffer**: Handles missed events when plugin is closed/reopened

### 2. Documentation Processing (2-Phase Pipeline)
- **Phase 1: Fetching** (content-fetcher.ts)
  - Multi-strategy fetching with fallbacks
  - Bundled content support
  - Unstyled content handling for Grafana docs
- **Phase 2: Parsing** (html-parser.ts → content-renderer.tsx)
  - HTML → React component tree conversion
  - Fail-fast error handling with detailed diagnostics
  - Interactive element extraction and configuration

### 3. Interactive Guide System (Layered Architecture)
- **Component Layer**: React components (InteractiveSection, InteractiveStep, etc.)
- **Hook Layer**: Business logic (interactive.hook, step-checker.hook)
- **Handler Layer**: Action execution (FocusHandler, ButtonHandler, etc.)
- **Manager Layer**: State coordination (InteractiveStateManager, SequentialRequirementsManager)
- **Utility Layer**: DOM operations (navigation-manager, element-validator, enhanced-selector)

### 4. Requirements & Objectives System

**Step Checking Priority** (`step-checker.hook.ts`):

1. **Check Objectives** (`data-objectives`)
   - If met → Auto-complete (`completionReason: objectives`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grafana/grafana-pathfinder-app](https://github.com/grafana/grafana-pathfinder-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
