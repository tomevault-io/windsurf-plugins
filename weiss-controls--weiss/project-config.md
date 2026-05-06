---
trigger: always_on
description: Conventions, patterns, and codebase context for AI-assisted development.
---

<!---
Conventions, patterns, and codebase context for AI-assisted development.
For architecture, configuration, and deployment details, see docs/src/.
-->

# WEISS — AI Development Context

WEISS is a web-based EPICS OPI designer and runtime viewer: engineers design display panels in a
browser, store them in Git repositories, and serve them with real-time EPICS PV data via WebSocket.

**License header** — required on every new source file:

```
// SPDX-License-Identifier: GPL-3.0-or-later
// Copyright (C) 2026 <Author(s) name(s)>
```

---

## High-Level Architecture

```
Browser (React + Vite)
  └─ REST  ──► FastAPI (weiss-api)   port 8000
  └─ WS    ──► epicsWS               port 8080   (proxied via NGINX as /ws/ in prod)
                └─ EPICS CA / PVA
```

Three services in Docker:

| Service               | Path               | Purpose                          |
| --------------------- | ------------------ | -------------------------------- |
| `weiss` / `weiss-dev` | `/` (Vite + NGINX) | Frontend SPA                     |
| `weiss-api`           | `backend/api/`     | FastAPI: auth, Git repo CRUD     |
| `weiss-epicsws`       | `backend/epicsWS/` | Python WebSocket bridge to EPICS |

---

## Frontend

**Stack:** React 19, TypeScript 5.8, Vite 7, MUI 7, react-rnd, react-router-dom 7, Plotly.js,
`@hey-api/openapi-ts` (generated API client).

**Path aliases** (configured in `vite.config.ts`):

- `@src` → `src/`
- `@components` → `src/components/`

### Entry Point

`src/main.tsx` bootstraps the app inside `BrowserRouter`. Routes:

- `/login` → `LoginPage`
- `/auth/callback` → `AuthCallback`
- `/` → `ProtectedRoute` → `App`

`ContextProvider` (`src/context/ContextProvider.tsx`) wraps the entire app and composes four context
providers.

### State Management (Context)

All state lives in React context — no Redux. Four context providers composed inside a single
`ContextProvider`:

| Context            | Hook               | What it owns                                                                                      |
| ------------------ | ------------------ | ------------------------------------------------------------------------------------------------- |
| `WidgetContext`    | `useWidgetManager` | All widgets on the canvas, selection, undo/redo, clipboard, grouping, import/export               |
| `UIContext`        | `useUIManager`     | Edit/runtime mode toggle, auth state, repo tree, file open/save, drag/pan flags                   |
| `EpicsWSContext`   | `useEpicsWS`       | WebSocket lifecycle, PV subscriptions, `pvState` cache, macro substitution map                    |
| `WSActionsContext` | `useEpicsWS`       | Exposes only `writePVValue`; separate context so write-only widgets don't re-render on PV updates |

#### `useWidgetManager` — mutation discipline

- `editorWidgets: Widget[]` — flat list; the grid is always `editorWidgets[0]` with
  `id === GRID_ID`.
- **All mutations go through** `updateEditorWidgetList(newWidgets, keepHistory)` — this is the
  single mutation point and pushes to the undo stack.
- `updateWidgetProperties(id, updates)` and `batchWidgetUpdate(multiUpdates)` are convenience
  wrappers around `updateEditorWidgetList`.
- Undo/redo via `undoStack`/`redoStack` (capped at `MAX_HISTORY`).
- `formatWdgToExport()` / `loadWidgets()` for serialization.

#### `useEpicsWS`

- `pvState: Record<pvName, PVData>` — reactive PV data fed to widget renders.
- `PVMap: Map<originalPV, substitutedPV>` — macro substitution, computed by `useWidgetManager`.
- `writePVValue(pvName, value)` — sends a write message; exposed via `WSActionsContext` so
  write-only widgets don't re-render on every PV update.

#### `useUIManager`

- `mode: "edit" | "runtime"` — drives whether widgets are interactive or editable.
- `user: User | null`, `isAuthenticated`, `isDeveloper` — auth state; developer role gates staging
  API.
- `reposTreeInfo` — fetched staging or deployment tree depending on role.
- Auto-saves to a staging repo file on property changes (debounced).

### Widget System

Every widget is a `WidgetDefinition` object:

```ts
export interface WidgetDefinition {
  widgetName: string; // registry key / serialization key
  widgetLabel: string; // palette display name
  widgetIcon?: WidgetIconType;
  component: React.ComponentType<WidgetUpdate>; // rendering component
  category: string; // palette grouping
  defaultProperties: WidgetProperties;
}
```

Runtime instances are `Widget` objects (stored in `editorWidgets`):

```ts
export interface Widget {
  id: string;
  widgetName: string;
  editableProperties: WidgetProperties; // subset of PROPERTY_SCHEMAS entries
  children?: Widget[]; // for groups / EmbeddedDisplay
  pvData?: PVData; // merged at render time only
  multiPvData?: Record<string, PVData>;
}
```

`WidgetProperties` is `Partial<typeof PROPERTY_SCHEMAS>` — each key maps to a `WidgetProperty<T>`.

#### Property System

All properties are defined in `src/types/widgetProperties.ts` via `PROPERTY_SCHEMAS`. Each property
has:

- `selType` — which editor control renders it (`"text"`, `"number"`, `"boolean"`, `"colorSel"`,
  `"select"`, `"strList"`, `"strRecord"`, `"repoFile"`, `"none"`)
- `label`, `value`, `category`, optional `options` and `limits`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [weiss-controls/weiss](https://github.com/weiss-controls/weiss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
