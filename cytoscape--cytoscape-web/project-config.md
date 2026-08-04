---
trigger: always_on
description: > Local context for `src/app-api/`. Read this before implementing any app API hook, core function, or event bus code.
---

# CLAUDE.md — App API Layer

> Local context for `src/app-api/`. Read this before implementing any app API hook, core function, or event bus code.

## Purpose

This directory is the **sole public API** for external apps consuming Cytoscape Web. It has two
access paths:

- **Module Federation** — React apps import `useXxxApi()` hooks from `cyweb/ElementApi` etc.
- **`window.CyWebApi`** — Vanilla JS consumers (browser extensions, LLM agent bridges) access the
  same operations via the global object assigned during app initialization.

Both paths execute the same domain logic, which lives entirely in `src/app-api/core/`.

## Directory Structure

```
src/app-api/
├── core/                        ← Framework-agnostic domain logic (no React)
│   ├── elementApi.ts
│   ├── networkApi.ts
│   ├── selectionApi.ts
│   ├── tableApi.ts
│   ├── visualStyleApi.ts
│   ├── layoutApi.ts            ← dispatches layout:started / layout:completed events directly
│   ├── viewportApi.ts
│   ├── exportApi.ts
│   ├── workspaceApi.ts         ← workspace state reads/writes (WorkspaceStore + NetworkSummaryStore)
│   ├── contextMenuApi.ts       ← context menu item registry (ContextMenuItemStore)
│   └── index.ts                 ← Assembles CyWebApi object; assigned to window.CyWebApi
├── event-bus/                   ← Typed event bus (Step 2, after Phase 1e)
│   ├── CyWebEvents.ts           ← CyWebEvents interface (8 event types + detail shapes)
│   ├── dispatchCyWebEvent.ts    ← Generic dispatch helper — sole place new CustomEvent() is called
│   └── initEventBus.ts          ← Zustand subscribeWithSelector → window.dispatchEvent
├── useElementApi.ts             ← React Hook: returns elementApi (thin wrapper)
├── useNetworkApi.ts
├── useSelectionApi.ts
├── useTableApi.ts
├── useVisualStyleApi.ts
├── useLayoutApi.ts
├── useViewportApi.ts
├── useExportApi.ts
├── useWorkspaceApi.ts           ← React Hook: returns workspaceApi (thin wrapper)
├── useCyWebEvent.ts             ← React Hook: window.addEventListener wrapper with cleanup
├── api_docs/
│   └── Api.md                   ← Behavioral documentation
├── types/
│   ├── ApiResult.ts             ← ApiResult<T>, ApiError, error code catalogs (ElementCodes, TableCodes, StyleCodes, AppCodes)
│   ├── AppContext.ts            ← AppContext, CyAppWithLifecycle
│   ├── ElementTypes.ts          ← Curated re-exports of public model types
│   └── index.ts                 ← Barrel export
└── index.ts                     ← Barrel export
```

## Core Principles

1. **Core logic is framework-agnostic** — All domain logic lives in `src/app-api/core/<domain>Api.ts`.
   These files use `useXxxStore.getState()` and never import React or call internal React hooks
   (`useCreateNode`, etc.).
2. **Hooks are thin wrappers** — `use<Domain>Api.ts` files return the corresponding core object.
   They contain no domain logic. `useElementApi = (): ElementApi => elementApi`.
3. **Wrap, never duplicate store coordination** — Core functions delegate to internal hooks where
   possible via `.getState()`. They do NOT call `useCreateNode()` etc. (React context required).
4. **Always return `ApiResult<T>`** — Never throw exceptions across the app API boundary.
5. **Validate before mutating** — Check store state existence before calling any store method.
6. **Hide `skipUndo`** — Hardcode to `false`; external apps must not corrupt the undo stack.
7. **No React imports in `core/`** — ESLint should flag any `import ... from 'react'` inside
   `src/app-api/core/`.
8. **`dispatchCyWebEvent` is the only dispatch site** — All 6 store-subscription-driven events go
   through `event-bus/dispatchCyWebEvent.ts`. Never call `window.dispatchEvent(new CustomEvent(...))`
   directly anywhere else.
9. **`initEventBus()` is called once after hydration** — `window.CyWebApi = CyWebApi` is assigned
   in `src/init.tsx` (before React renders). `initEventBus()` and `cywebapi:ready` are called in
   `src/features/AppShell.tsx` immediately after `setWorkspace(workspace)` completes, so
   subscriptions are never active during the IndexedDB → store hydration transition and no
   spurious `network:created` / `network:switched` events fire on startup.
10. **Layout events come from `core/layoutApi.ts`** — Not from store subscriptions. `layout:started`
    fires before `LayoutStore.setIsRunning(true)`, `layout:completed` fires inside the layout
    promise resolution. Errors do NOT dispatch `layout:completed`.

## Two-Layer Pattern

### Layer 1: Core function (framework-agnostic)

```typescript
// src/app-api/core/elementApi.ts
import { useNetworkStore } from '../../data/hooks/stores/NetworkStore'
import { ok, fail, AppCodes } from '../types'
import type { ElementApi } from '../types'

export const elementApi: ElementApi = {
  createNode(networkId, position, options): ApiResult<{ nodeId: IdType }> {
    try {
      // 1. Validate inputs using .getState() — no React context needed
      const network = useNetworkStore.getState().networks.get(networkId)
      if (!network)
        return fail(AppCodes.NETWORK_NOT_FOUND, networkId)
      // 2. Coordinate stores directly
      // ...
      return ok({ nodeId })
    } catch (e) {
      // 3. Catch exceptions → ApiFailure

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cytoscape/cytoscape-web](https://github.com/cytoscape/cytoscape-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
