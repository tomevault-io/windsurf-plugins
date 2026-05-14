---
trigger: always_on
description: This file provides guidance to Claude Code when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Overview

BESSER Web Modeling Editor (WME) is the frontend for the BESSER low-code platform. It provides a browser-based visual editor for creating UML diagrams, GUI designs, agent models, quantum circuits, and more. The editor communicates with a Python/FastAPI backend for code generation, validation, and deployment.

- **Live**: https://editor.besser-pearl.org
- **Backend repo**: https://github.com/BESSER-PEARL/BESSER
- **This repo is vendored** into the backend as a git submodule at `besser/utilities/web_modeling_editor/frontend`

## Monorepo Structure

This is an npm workspaces monorepo with 3 packages:

| Package | Status | Purpose |
|---------|--------|---------|
| `packages/webapp` | **Active** | Main React SPA (Vite + React 18 + Tailwind + Radix UI) |
| `packages/editor` | **Active** | Core diagramming engine, published as `@besser/wme` on npm |
| `packages/server` | **Active** | Express server for standalone hosting (serves built webapp) |

Almost all feature work happens in `webapp` and `editor`.

## Essential Commands

```bash
# Install all dependencies (run from monorepo root)
npm install

# Development (starts Vite dev server on http://localhost:8080)
# Requires BESSER backend running at http://localhost:9000
npm run dev

# Build for production
npm run build              # Builds webapp + server
npm run build:webapp      # webapp only
npm run build:local        # Build with localhost backend URLs

# Testing
npm run test               # Vitest unit tests (webapp)
npm run test:e2e           # Playwright E2E tests
npm run test:e2e:ui        # E2E with interactive UI

# Linting & formatting
npm run lint               # ESLint (webapp + server)
npm run prettier:check     # Check formatting
npm run prettier:write     # Auto-format

# Standalone server (after building)
npm run start:server       # Express on http://localhost:8080
```

**Node requirement**: >= 20.0.0

## Architecture Overview

### Tech Stack
- **Build**: Vite 7 (webapp), Webpack (server, editor)
- **Framework**: React 18.2 + React Router 6
- **State**: Redux Toolkit (single `workspaceSlice` + `errorManagementSlice`)
- **UI**: Radix UI primitives + Tailwind CSS (class-based dark mode)
- **Editors**: ApollonEditor (UML), GrapesJS (GUI no-code), custom (quantum circuits)
- **Testing**: Vitest + jsdom (unit), Playwright (E2E)
- **TypeScript**: 5.6, strict mode, ES2021 target

### Source Layout (webapp)

```
packages/webapp/src/main/
├── app/                        # Shell, routing, Redux store
│   ├── application.tsx         # Root: routes, providers, lazy dialogs
│   ├── shell/                  # TopBar, Sidebar, menus
│   └── store/                  # store.ts, workspaceSlice.ts, hooks.ts
├── features/                   # Feature modules (isolated)
│   ├── editors/                # EditorView + UML/GUI/quantum editors
│   ├── generation/             # Code generation dialogs & hooks
│   ├── deploy/                 # Render deployment
│   ├── github/                 # GitHub OAuth
│   ├── import/                 # Import dialogs
│   ├── export/                 # Export dialogs
│   ├── assistant/              # AI assistant widget + services
│   ├── agent-config/           # Agent configuration panels
│   ├── project/                # Project hub, settings, templates
│   └── onboarding/             # Tutorial flow
├── shared/                     # Cross-feature code
│   ├── api/                    # ApiClient (centralized HTTP)
│   ├── components/             # Reusable UI components
│   ├── constants/              # Environment vars, localStorage keys
│   ├── hooks/                  # Shared React hooks
│   ├── services/               # Storage, validation, analytics
│   ├── types/                  # TypeScript types (BesserProject, etc.)
│   └── utils/                  # Pure utilities
└── templates/                  # Starter project templates
```

### Editor Package (packages/editor)

The diagramming engine, published as `@besser/wme`. Contains:

```
packages/editor/src/main/
├── apollon-editor.ts           # Public API (ApollonEditor class)
├── packages/                   # Diagram-specific implementations
│   ├── uml-class-diagram/      # Class diagram elements
│   ├── uml-object-diagram/     # Object diagram elements
│   ├── uml-state-diagram/      # State machine elements
│   ├── agent-state-diagram/    # Agent diagram elements
│   ├── flowchart/              # Flowchart elements
│   ├── bpmn/                   # BPMN elements
│   ├── common/                 # Shared element types
│   ├── components.ts           # Element type → React component registry
│   ├── uml-elements.ts         # Element type → model class registry
│   ├── compose-preview.ts      # Element type → palette preview registry
│   ├── popups.ts               # Element type → property popup registry
│   └── diagram-type.ts         # Supported diagram types enum
├── services/                   # Domain logic (CRUD, undo, layout, collaboration)
├── components/                 # Canvas, sidebar, event listeners
└── i18n/                       # Translations (en, de, etc.)
```

## Key Patterns

### Feature Isolation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BESSER-PEARL/BESSER-Web-Modeling-Editor](https://github.com/BESSER-PEARL/BESSER-Web-Modeling-Editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
