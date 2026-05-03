---
trigger: always_on
description: This file provides guidance for Claude Code when working with this repository.
---

# CLAUDE.md

This file provides guidance for Claude Code when working with this repository.

## Project Overview

WaveSpeed Desktop is an Electron-based cross-platform desktop application that provides a playground interface for [WaveSpeedAI](https://wavespeed.ai) models. It allows users to browse models, run predictions, view their history, and manage saved assets.

**Workflow** is a node-based visual editor (under `src/workflow/`) for chaining WaveSpeed AI Task nodes, free-tool nodes, and I/O nodes. Workflows are persisted via the Electron main process (sql.js DB), with execution and per-node history. Workflows can run in Electron (main process) or in the browser (in-process executor using apiClient + free-tool runners when workflow IPC is unavailable). Cost is informational only (no estimate UI or budget blocking).

**Z-Image** is the local image generation flow backed by stable-diffusion.cpp with model and auxiliary downloads, progress reporting, and log streaming.

## Tech Stack

- **Electron** with **electron-vite** for the desktop framework
- **React 18** + **TypeScript** for the UI
- **Tailwind CSS** + **shadcn/ui** for styling
- **Zustand** for state management
- **Axios** for HTTP requests

## Project Structure

```
wavespeed-desktop/
├── electron/              # Electron main process files
│   ├── main.ts           # Main process entry point
│   ├── preload.ts        # Preload script for IPC bridge
│   └── workflow/         # Workflow module (sql.js DB, node registry, IPC handlers)
├── src/
│   ├── api/
│   │   └── client.ts     # WaveSpeedAI API client (base URL, auth, methods)
│   ├── components/
│   │   ├── layout/       # Sidebar, Layout components
│   │   ├── playground/   # DynamicForm, FileUpload, OutputDisplay, MaskEditor, etc.
│   │   ├── shared/       # ApiKeyRequired and other shared components
│   │   └── ui/           # shadcn/ui components (Button, Card, etc.)
│   ├── hooks/            # Custom React hooks (useToast, useUpscalerWorker, useMultiPhaseProgress)
│   ├── i18n/             # Internationalization with react-i18next
│   │   └── locales/      # 18 language locale files
│   ├── lib/              # Utilities (cn, fuzzySearch, schemaUtils, maskUtils)
│   ├── pages/            # Page components (ModelsPage, PlaygroundPage, FreeToolsPage, etc.)
│   ├── stores/           # Zustand stores (apiKeyStore, modelsStore, settingsStore, etc.)
│   ├── types/            # TypeScript type definitions
│   ├── workflow/         # Workflow feature (node-based editor)
│   │   ├── WorkflowPage.tsx
│   │   ├── components/   # WorkflowList, panels (NodeConfig, Results, Cost, Settings), canvas (WorkflowCanvas, NodePalette, CustomNode, CustomEdge, AnnotationNode, etc.)
│   │   ├── stores/       # workflow.store, execution.store, ui.store
│   │   ├── types/        # workflow, node-defs, execution, ipc
│   │   ├── ipc/          # ipc-client.ts (invoke workflow/execution/models/cost/history IPC)
│   │   ├── hooks/        # useUndoRedo, useFreeToolListener
│   │   ├── browser/     # run-in-browser.ts (execution), workflow-api.ts, workflow-storage.ts
│   │   └── lib/         # free-tool-runner, model-converter, outputDisplay, topological
│   └── workers/          # Web Workers (upscaler, backgroundRemover, imageEraser, faceEnhancer, segmentAnything, ffmpeg)
├── .github/workflows/    # GitHub Actions for CI/CD
│   ├── build.yml         # Build on push/tag/PR
│   └── nightly.yml       # Nightly builds
└── build/                # Build resources (icons, etc.)
```

## Key Files

- **`src/api/client.ts`**: API client with all WaveSpeedAI endpoints
- **`src/stores/apiKeyStore.ts`**: API key persistence and validation (electron-store + localStorage fallback)
- **`src/stores/modelsStore.ts`**: Model list caching, filtering, and sorting (supports sort_order/popularity)
- **`src/stores/playgroundStore.ts`**: Multi-tab playground state management
- **`src/stores/templateStore.ts`**: Template CRUD operations with localStorage persistence
- **`src/stores/themeStore.ts`**: Theme management (auto/dark/light) with system preference detection
- **`src/stores/assetsStore.ts`**: Asset management (save, delete, tags, favorites, filtering)
- **`src/stores/settingsStore.ts`**: App settings (e.g. download timeout) with localStorage persistence
- **`src/workflow/WorkflowPage.tsx`**: Workflow feature entry; workflow list + canvas or list-only view
- **`src/workflow/stores/workflow.store.ts`**: Workflow CRUD state (nodes, edges, current workflow id)
- **`src/workflow/stores/execution.store.ts`**: Execution state (running, results, history, progress)
- **`src/workflow/stores/ui.store.ts`**: Workflow UI state (selected node, panels, add-node palette)
- **`src/workflow/components/panels/NodeConfigPanel.tsx`**: Model selector for AI Task nodes; categories sorted by popularity (model count), recent models, search
- **`src/workflow/components/panels/ResultsPanel.tsx`**: Execution results and per-node history; shows lastResults when history is empty
- **`src/workflow/components/panels/CostPanel.tsx`**: Cost display (informational; no estimate UI or budget blocking)
- **`src/workflow/components/panels/SettingsPanel.tsx`**: Workflow settings (API keys, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WaveSpeedAI/wavespeed-desktop](https://github.com/WaveSpeedAI/wavespeed-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
