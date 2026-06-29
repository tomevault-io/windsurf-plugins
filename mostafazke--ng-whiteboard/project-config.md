---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Serve demo app
yarn start

# Build library + demo
yarn build
yarn build:lib   # library only (ng-packagr → dist/ng-whiteboard)
yarn build:demo  # demo PWA only

# Test
yarn test:lib    # library tests with coverage (watch mode)
yarn test:demo   # demo tests with coverage (watch mode)
yarn test:ci     # library tests in CI mode (lcov coverage)

# Lint (via Nx)
nx run ng-whiteboard:lint
nx run demo:lint

# Conventional commits
yarn cm
```

Running a single test file:

```bash
npx nx test ng-whiteboard --testFile=projects/ng-whiteboard/src/lib/core/tools/pen/pen.tool.spec.ts
```

## Architecture

This is an **Nx monorepo** with two Angular 17 projects:

- `projects/ng-whiteboard` — publishable Angular library (npm: `ng-whiteboard`)
- `projects/demo` — standalone PWA demo deployed to GitHub Pages

The library renders entirely via **SVG** (no Canvas 2D). It uses Angular standalone components, `inject()`, Signals, and `OnPush` change detection throughout.

### Library structure (`projects/ng-whiteboard/src/lib/`)

**Entry points:**

- `NgWhiteboardComponent` (`ng-whiteboard.component.ts`) — root component `<ng-whiteboard>`, provides all services, hosts `<ng-whiteboard-canvas>` and `<wb-context-menu>`
- `NgWhiteboardService` (`ng-whiteboard.service.ts`) — public facade for consumers; delegates to `ApiService` for the active board
- `src/index.ts` — public API surface

**Multi-instance pattern:** `InstanceService` (root) holds a `Map<boardId, ApiService>`. Call `whiteboardService.setActiveBoard(boardId)` to target a specific board instance.

**Core subsystems in `core/`:**

| Directory                  | Key service/class                                                                      | Responsibility                                                                                         |
| -------------------------- | -------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| `api/`                     | `ApiService`                                                                           | Internal aggregate API per board instance                                                              |
| `canvas/`                  | `CanvasService`, `InstanceService`                                                     | SVG canvas management, board registry                                                                  |
| `components/canvas/`       | `WhiteboardCanvasComponent`                                                            | SVG rendering surface                                                                                  |
| `components/context-menu/` | `ContextMenuComponent`, `ContextMenuService`                                           | Right-click menu                                                                                       |
| `config/`                  | `ConfigService`                                                                        | `WhiteboardConfig` signal management                                                                   |
| `elements/`                | `ElementsService`, `SelectionService`, `LayerManagementService`, `ArrowBindingService` | Element CRUD, selection, z-order, arrow smart binding                                                  |
| `event-bus/`               | `EventBusService`                                                                      | Typed event bus — RxJS `Subject` + Angular Signals dual-emission for all `WhiteboardEvent` enum values |
| `history/`                 | `HistoryService`                                                                       | Undo/redo stack                                                                                        |
| `input/`                   | `IOService`, `KeyboardShortcutService`, `ClipboardService`, `DragDropService`          | Pointer events, shortcuts, clipboard, drag-and-drop                                                    |
| `tools/`                   | `BaseTool`, `ToolsService`, `ToolFactory`                                              | Strategy pattern — each tool extends `BaseTool` and implements `handlePointerDown/Move/Up`             |
| `viewport/`                | `ZoomService`, `PanService`, `WheelHandlerService`                                     | Zoom, pan, wheel input                                                                                 |
| `svg/`                     | `SvgService`, `SvgDirective`                                                           | SVG utilities and export                                                                               |
| `types/`                   | `elements.ts`, `tools.ts`, `config.ts`, `events.ts`                                    | All TypeScript types and enums                                                                         |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mostafazke/ng-whiteboard](https://github.com/mostafazke/ng-whiteboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
