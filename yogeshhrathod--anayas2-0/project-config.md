---
trigger: always_on
description: - Desktop application built with Electron 28, Vite 5, React 18, and TypeScript for professional REST API testing and development.@package.json#1-61 @vite.config.ts#1-47 @README.md#1-128
---

# Cursor Rules

## Project Summary
- Desktop application built with Electron 28, Vite 5, React 18, and TypeScript for professional REST API testing and development.@package.json#1-61 @vite.config.ts#1-47 @README.md#1-128
- Electron main process initializes a JSON-backed datastore, registers IPC handlers, and loads the renderer. Node integration is disabled; preload exposes a typed bridge via `contextBridge`.@electron/main.ts#1-79 @electron/database/json-db.ts#27-65 @electron/ipc/index.ts#1-1 @electron/preload.ts#1-150
- Renderer is a React/Tailwind UI living under `src/` with Zustand for state management and Monaco-based editors for JSON templates.@src/main.tsx#1-10 @README.md#92-128

## Development Workflow
1. Install dependencies: `npm install`.
2. Run desktop dev mode: `npm run electron:dev` (starts Vite + Electron with hot reload).@package.json#8-17 @README.md#63-88
3. Type-check: `npm run type-check` before shipping changes.@package.json#8-18
4. Production build: `npm run build` (or platform-specific `build:mac|win|linux`).@package.json#8-18 @README.md#78-90

## Coding Guidelines
- Keep TypeScript strictness; prefer typed APIs in preload and IPC layers. Extend interfaces in `electron/preload.ts` when exposing new functionality.@electron/preload.ts#1-150
- All renderer-to-main communication must go through `window.electronAPI` helpers created in preload. Do **not** enable Node integration or access `ipcRenderer` directly from React components.@electron/preload.ts#61-150
- Register IPC handlers in `electron/ipc/handlers.ts` and export updates via `electron/ipc/index.ts`. Ensure handlers validate inputs and surface errors with descriptive logs.@electron/ipc/handlers.ts#22-31 @electron/ipc/index.ts#1-1 @electron/services/logger.ts#1-47
- Use the Winston logger for background services; avoid `console.log` in production paths. Configure module-specific loggers via `createLogger` helper.@electron/services/logger.ts#1-47
- Persist application state through the JSON DB helpers in `electron/database`. Add schema migrations or default seeding within `initDatabase` when required.@electron/database/json-db.ts#27-65
- Maintain strict separation between UI state (Zustand) and IPC contracts. Update shared types under `src/types/` when introducing new data structures.@README.md#92-128

## UI Conventions
- TailwindCSS governs styling with shadcn/ui primitives; respect existing utility classes and component patterns.
- Place shared UI pieces under `src/components/` and domain screens under `src/pages/`.
- Favor composition and hooks aligned with current store setup in `src/store/useStore.ts`.

## Feature Tracking & Spec-Driven Development

This project uses a **spec-driven development** workflow. All new features and bugs MUST follow this process:

### For Features:
1. **Create Feature**: Use `./scripts/create-feature.sh <feature-name>` to create a new feature in `specs/XXX-feature-name/`
2. **Write Spec**: Complete `spec.md` with user stories, acceptance criteria, and technical requirements
3. **Create Plan**: Complete `plan.md` with architecture decisions and implementation phases
4. **Break Down Tasks**: Complete `tasks.md` with specific, actionable tasks
5. **Implement**: Follow tasks.md, updating task status as you progress

### For Bugs:
1. **Create Bug**: Use `./scripts/create-bug.sh <bug-name>` to create a new bug in `specs/bug-XXX-bug-name/`
2. **Write Bug Report**: Complete `spec.md` with reproduction steps, expected/actual behavior, and environment details
3. **Create Fix Plan**: Complete `plan.md` with root cause analysis and fix strategy
4. **Break Down Fix Tasks**: Complete `tasks.md` with specific fix tasks and verification steps
5. **Implement Fix**: Follow tasks.md, updating bug status (reported → investigating → fixing → testing → resolved)

**ALWAYS** reference the feature/bug's spec.md, plan.md, and tasks.md when implementing. See `.cursor/rules/feature-tracking.mdc` for detailed workflow rules.

Link features to phases in `plan-timeline.md` using the `Phase` field in spec.md. Bug fixes can be linked to maintenance phases.

## Goal-Aware Development Workflow

**CRITICAL**: Before implementing any feature, ALWAYS follow this workflow. This ensures performance-first development and goal alignment.

### Step 1: SPEC (Do NOT write any code yet)

1. **Read context files FIRST**:
   - `/ai-context/project-goal.md` - Understand project goals and performance targets
   - `/ai-context/architecture.md` - Understand architecture patterns and performance strategies
   
2. **Create/update `spec.md`** in `specs/XXX-feature-name/`:
   - Include **Goal Alignment Summary** (how feature supports performance-first mission)
   - Include **Performance Impact Analysis** (memory, bundle size, load time estimates)
   - Include **Success Criteria** with performance metrics
   - Include **Constraints** (performance budgets, lazy loading requirements)
   - List any **unclear points** to confirm

3. **Wait for approval** before proceeding to Step 2.

### Step 2: PLAN (No code yet)

1. **Read context files**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yogeshhrathod) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
