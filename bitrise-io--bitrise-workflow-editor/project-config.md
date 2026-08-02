---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

@./node_modules/@bitrise/bitkit-v2/AGENTS.md

## Project Overview

Bitrise Workflow Editor — a React + Go application for editing CI/CD workflow configurations (bitrise.yml). Runs as a Bitrise CLI plugin (default) or as a website integrated with the Bitrise monolith. Transitioning from AngularJS to React.

## Common Commands

```bash
npm start                # Dev server + local Go API on port 4000
npm run start:website    # Dev server in website mode (requires monolith running on :3000)
npm run build            # Vite production build
npm run lint             # ESLint (cached)
npm run lint:fix         # ESLint autofix
npm test                 # Jest unit tests
npm test -- --testPathPattern="path/to/file"  # Run single test file
npm run test:smoke       # Playwright E2E tests
npm run storybook        # Storybook on port 6006
```

**Go API server:**
```bash
go vet ./...             # Vet Go code
go test ./...            # Go tests
```

**Setup:** `bitrise run setup` (installs Node + Go deps)

## Architecture

### Frontend (`source/javascripts/`)

- **Framework:** React 18 + TypeScript (strict mode), built with Vite
- **UI:** `@bitrise/bitkit-v2` (new, Chakra UI v3) for new components; `@bitrise/bitkit` (legacy, Chakra UI v2) still present but being replaced — use v2 for all new work, and migrate v1 components to v2 in any file you touch
- **State:** Zustand — `BitriseYmlStore` is the central store holding the YAML document
- **Data fetching:** TanStack React Query
- **Routing:** wouter (lazy-loaded pages)
- **YAML editing:** Monaco Editor + monaco-yaml + custom `@bitrise/languageserver`
- **Graph visualization:** XYFlow + dagre (pipeline/workflow graphs)
- **Drag & drop:** dnd-kit
- **Path alias:** `@/` → `source/javascripts/`

### Key directories

```
source/javascripts/
  core/
    api/           # API clients (BitriseYmlApi, StepApi, EnvVarsApi, etc.)
    stores/        # Zustand stores (BitriseYmlStore is the main one)
    models/        # TypeScript types for BitriseYml, Step, Workflow, etc.
    services/      # Domain logic (StepService, PipelineService, etc.)
  hooks/           # React hooks (useCiConfig, useSecrets, useFeatureFlag, etc.)
  components/      # Shared + unified-editor components
  pages/           # WorkflowsPage, PipelinesPage, TriggersPage, etc.

apiserver/         # Go HTTP server (Gorilla Mux), serves API + embedded static assets
cmd/               # Go CLI (Cobra)
spec/              # Test files (Jest unit + Playwright E2E)
```

### Patterns

- **`core/` is framework-agnostic** — no React or DOM dependencies. Pure TypeScript only
  - **`models/`** — internal application types used throughout the app
  - **`api/`** — API client functions that work with DTOs and map them to internal models. Consumed by services and hooks, never called directly from components
  - **`services/`** — business logic operating on models. Must have thorough unit tests covering happy paths, edge cases, error conditions, and different YAML formats where applicable
  - **`stores/`** — Zustand stores (mainly `BitriseYmlStore`). Coordinate state across the app
- **YAML preservation:** Service functions that modify YAML must not make unnecessary changes or reorder existing fields — only touch what's needed
- **Component architecture:** hooks manage API calls and local state; components focus on rendering
- **Two modes:** `MODE=CLI` (plugin, default) and `MODE=WEBSITE` — runtime behavior branches via `PageProps`/`RuntimeUtils` and environment checks

### Service Conventions

- All services are **pure functions** exported via `export default { ... }`, not classes
- **Mutation pattern:** services mutate YAML via `updateBitriseYmlDocument(({doc}) => { ...; return doc })` — the store clones the document before calling, so services mutate `doc` directly
- **Validation pattern:** `getXOrThrowError(id, doc)` before any mutation to ensure the target exists
- **Validate functions** return `string | boolean` — `true` on success, error message string on failure
- Services **never import React** — they live in `core/` which is framework-agnostic
- **Dependency direction:** `WorkflowService` and `StepService` are foundational (no service deps); others build on top (`PipelineService` → `WorkflowService` + `StepService`, etc.)
- **Cross-service operations:** Some user actions (e.g., deleting a workflow) touch multiple services (removal + trigger cleanup + env var cleanup). There's no explicit orchestrator — the store or calling code coordinates these calls. Be aware of cascading effects when modifying service functions

### Hook Conventions

- **Store selectors** are thin hooks wrapping `useBitriseYmlStore` with `useShallow` (e.g., `useWorkflows`, `useContainers`)
- **Data fetching hooks** use TanStack React Query (`useQuery`/`useMutation`) with proper `staleTime`/`gcTime`
- Hooks should be **thin wrappers** — delegate business logic to services rather than implementing it inline

### Page Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bitrise-io/bitrise-workflow-editor](https://github.com/bitrise-io/bitrise-workflow-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
