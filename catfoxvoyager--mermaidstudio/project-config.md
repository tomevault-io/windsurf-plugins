---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

The canonical package manager is **pnpm 10** (`pnpm-lock.yaml` is committed); `npm >= 10` is also accepted. Either works for the commands below.

```bash
# Development
npm run dev              # Start Vite dev server on port 5173
npm run build            # Production build (tsc type-check + vite build)
npm run build:analyze    # Build with rollup-plugin-visualizer (emits dist/stats.html)
npm run preview          # Preview production build locally
npm run benchmark        # Run scripts/benchmark.mjs

# Code Quality
npm run lint             # ESLint check (eslint src --max-warnings 999)
npm run lint:fix         # ESLint auto-fix
npm run type-check       # TypeScript type check (tsc --noEmit)
npm run format           # Prettier write on src/**/*.{ts,tsx}
npm run format:check     # Prettier check (CI gate)

# Testing
npm test                 # Unit tests (Vitest)
npm run test:ui          # Vitest UI mode
npm run test:coverage    # Coverage report (thresholds: lines 65 / fn 64 / branches 59 / stmt 62)
npm run test:e2e         # Playwright E2E tests
npm run test:e2e:ui      # Playwright UI mode
npm run test:e2e:headed  # Playwright headed mode
npm run test:e2e:debug   # Playwright debug mode
npm run test:e2e:report  # Show Playwright HTML report
npm run test:lighthouse  # Lighthouse CI (expects dev server on :5173)
npm run e2e:install      # Install Playwright browsers
npm run e2e:report       # Generate consolidated E2E report (scripts/generate-e2e-report.js)

# Documentation
npm run docs             # Generate documentation (scripts/generate-docs.cjs)
npm run docs:check       # Check documentation quality
npm run docs:test        # Test documentation (scripts/test-docs.cjs)
```

## Project Architecture

**MermaidStudio** is a React 19 + TypeScript Vite application for editing Mermaid diagrams. It runs **entirely client-side** — there is no backend. Diagrams, settings, and AI keys persist to IndexedDB / localStorage in the browser, and AI inference runs in-browser via WebGPU. Features include a code editor (CodeMirror 6), live preview (Mermaid.js), visual editor, and an AI assistant.

### Directory Structure

```
src/
├── components/          # React components organized by feature
│   ├── ai/             # AI panel, system prompt
│   ├── editor/         # CodeMirror editor, tabs, workspace
│   ├── modals/         # Modal dialogs (diagram, settings, tools)
│   ├── preview/        # Diagram preview and style panels
│   ├── visual/         # Drag-and-drop visual editor
│   ├── sidebar/        # File browser sidebar
│   └── shared/         # Reusable UI (Modal, Toast, ContextMenu)
├── lib/mermaid/        # Mermaid.js integration layer
├── services/
│   ├── ai/             # AI provider implementation (WebGPU/MLC only)
│   └── storage/        # IndexedDB database wrapper
├── hooks/
│   ├── ai/             # AI-specific hooks (useAIChat, useAISend, useAISettings)
│   └── app/            # App state hooks (useAppState, useModalState)
├── i18n/               # i18next translations (en, fr)
├── constants/          # Themes, templates, theme derivation
├── types/              # TypeScript type definitions
└── utils/              # Utility functions (logger, encryption, sanitization, validation)
```

### Path Aliases (tsconfig.app.json)

Use these imports instead of relative paths:
- `@/components/*` → `src/components/*`
- `@/lib/*` → `src/lib/*`
- `@/services/*` → `src/services/*`
- `@/types` → `src/types`
- `@/hooks` → `src/hooks`
- `@/constants` → `src/constants`
- `@/utils` → `src/utils`
- Feature shortcuts: `@/ai`, `@/editor`, `@/preview`, `@/sidebar`, `@/visual`

### Component Organization

Components are organized by **feature domain**, not by type. Each feature directory may contain:
- The main component file
- `__tests__/` directory for co-located tests
- Sub-components when appropriate

**Example**: `src/components/preview/PreviewPanel.tsx` has tests in `src/components/preview/__tests__/PreviewPanel.test.tsx`

### State Management Pattern

The app uses **custom hooks for state**, not a state library. State is composed in `App.tsx` and threaded down through `AppLayout` and `ModalProvider` via props. Key hooks:

- `useTabs` (`hooks/useTabs.ts`) - Multi-tab diagram management with active tab tracking
- `useTheme` (`hooks/useTheme.ts`) - Dark/light theme with persistence
- `useToast` (`hooks/useToast.ts`) - Toast notifications via Radix UI
- `useKeyboardShortcuts` (`hooks/useKeyboardShortcuts.ts`) - Global keyboard shortcuts
- `useModalManager` (`hooks/useModalManager.ts`) - Modal open/close state (mutual exclusion)
- `useAIChat` (`hooks/ai/useAIChat.ts`) - AI chat history and responses

App-level orchestration hooks live in `hooks/app/` (`useAppState`, `useModalState`, etc.). For complex state, prefer colocating new state in a focused hook under `src/hooks/` rather than expanding the `AppLayout`/`ModalProvider` prop lists.

### Mermaid Integration

The `src/lib/mermaid/` directory wraps Mermaid.js:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CatFoxVoyager/MermaidStudio](https://github.com/CatFoxVoyager/MermaidStudio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
