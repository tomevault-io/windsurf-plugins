---
trigger: always_on
description: **brs-engine** is a TypeScript-based interpreter for Roku's BrightScript language that runs on browsers, Node.js, and Electron. It simulates Roku's runtime environment, including the Draw 2D API and SceneGraph framework, enabling Roku app development and testing outside of Roku hardware.
---

# BrightScript Engine - AI Agent Instructions

## Project Overview

**brs-engine** is a TypeScript-based interpreter for Roku's BrightScript language that runs on browsers, Node.js, and Electron. It simulates Roku's runtime environment, including the Draw 2D API and SceneGraph framework, enabling Roku app development and testing outside of Roku hardware.

**Key Architecture:**
- **Monorepo structure** with three packages: `packages/browser` (web/Electron) published as `brs-engine`, `packages/node` (CLI/server) published as `brs-node`, and `packages/scenegraph` published as `brs-scenegraph`
- **Web Worker architecture**: Browser package runs interpreter in a Web Worker (`brs.worker.js`) with API library (`brs.api.js`) for host communication
- **Lexer → Parser → Interpreter pipeline**: Code flows through `src/core/lexer` → `src/core/parser` → `src/core/interpreter`
- **Component System**: BrightScript objects live in `src/core/brsTypes/components`; SceneGraph nodes live in `src/extensions/scenegraph/nodes`
- **Extension System**: SceneGraph (and other future features) are plug-in extensions implementing `BrsExtension` interface (`src/core/extensions.ts`); registered with `registerExtension()`
- **Device Simulation**: `BrsDevice` (in `src/core/device/`) maintains simulated Roku device state (registry, file system, device info)
- **Roku OS Version**: Currently synchronized with Roku OS 15.x features and APIs

## Recent Major Improvements

### SceneGraph Extension Architecture (v2.x)
- **Third package**: `packages/scenegraph` (published as `brs-scenegraph` v0.1.0+) is a standalone addon that owns the XML component parser, `RoSGScreen`, all nodes, and Task execution helpers
- **Extension system**: `BrsExtension` interface (`src/core/extensions.ts`) is the plug-in contract; `registerExtension()` wires extensions into every new interpreter instance without forking the core
- **BrightScriptExtension class**: The entry point for the SceneGraph addon (`src/extensions/scenegraph/index.ts`), implements `BrsExtension`
- **SGRoot singleton** (`src/extensions/scenegraph/SGRoot.ts`): Holds the active scene, `m.global`, focus, threads, timers, and animations
- **SupportedExtension enum**: `SupportedExtension.SceneGraph` (`"brs-scenegraph"`), `SDK1` and `BrightSign` extensions are planned

### SceneGraph Rendezvous & Threads (v2.1.0)
- **Real Rendezvous**: SceneGraph now implements thread updates similar to Roku's Rendezvous pattern for safe cross-thread field access
- **Multiple roMessagePort in Main thread**: Tasks and Main thread can each own multiple ports
- **Task debugger support**: Task threads can be paused at breakpoints via the MicroDebugger

### RoSGNode Refactoring & Field System
- **Abstract base class**: `RoSGNode` is abstract; all nodes must extend either `RoSGNode` or the concrete `Node` class
- **Field type validation**: `Field.canAcceptValue()` validates typed arrays (`intarray`, `floatarray`, `boolarray`, `stringarray`, `colorarray`, `timearray`) and converts string values to the correct type
- **Field aliases**: Nodes support multiple comma-separated field aliases; aliases fire observers correctly and support child change propagation
- **System fields are protected**: Cannot be removed via `removeField()` or added via `setFields()`; use `addFields()` for new fields
- **setValue vs setValueSilent**: `setValue()` triggers observers; `setValueSilent()` does not (used during initialization)
- **`setValue()` signature**: Does NOT create a field if it does not already exist (prevents accidental field creation on assignment)

### New SceneGraph Nodes (v2.0 – v2.1)
- **Animation system**: `Animation`, `ParallelAnimation`, `SequentialAnimation`, `FloatFieldInterpolator`, `ColorFieldInterpolator`, `Vector2DFieldInterpolator`
- **Panel nodes**: `PanelSet`, `Panel`, `ListPanel`, `GridPanel`, `OverhangPanelSetScene`, `Overhang`
- **Standard dialogs**: `StandardKeyboardDialog`, `StandardProgressDialog`, `StdDlgContentArea`, `StdDlgProgressItem`, `StdDlgTitleArea`
- **Input nodes**: `PinPad`, `VoiceTextEditBox`, `ScrollableText`, `ScrollingLabel`
- **Grid nodes**: `PosterGrid`, `MaskGroup`
- **Other**: `InfoPane`, `RSGPalette`, `ChannelStore`

### Content Handling Pattern
- **Standardized content caching**: All list/grid nodes follow the `refreshContent()` pattern
- **setValue override**: Content fields processed in `setValue()`, triggering `refreshContent()`
- **Performance optimization**: Content cached once, never re-fetched per frame

### File System Improvements
- **Case preservation**: Writeable volumes (`tmp:`, `cachefs:`) preserve original case
- **Shared volumes**: `tmp:` and `cachefs:` are shared among threads
- **Pinned `@zenfs/core` v2.5.0**: Using synchronous config for reliability

### Type System Enhancements
- **Uninitialized validation**: Raises type mismatch error when passing `Uninitialized` to non-dynamic function parameters
- **Double support**: Added `d` flag to `ParseJson()` for parsing to `double` type

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lvcabral/brs-engine](https://github.com/lvcabral/brs-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
