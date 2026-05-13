---
trigger: always_on
description: Loggy is a desktop application for viewing AWS CloudWatch logs. Built with Tauri (Rust backend) + React/TypeScript frontend.
---

# AI Agent Instructions

## Project Overview

Loggy is a desktop application for viewing AWS CloudWatch logs. Built with Tauri (Rust backend) + React/TypeScript frontend.

See `docs/DESIGN.md` for full architecture documentation.

## Tech Stack

- **Backend**: Tauri 2.x, Rust, AWS SDK for Rust
- **Frontend**: React 19, TypeScript, Zustand, react-window v2, Fuse.js, react-markdown, Tailwind CSS v4
- **Build**: Vite, trunk (formatting/linting)

## Key Files

- `src-tauri/src/lib.rs` - Rust backend with AWS CloudWatch integration
- `src/stores/groupStore.ts` - Editor group layout tree, split/merge operations (zustand/persist)
- `src/stores/workspaceStore.ts` - Panel state, merged view computation, correlation
- `src/stores/panelSlice.ts` - Per-panel state factory (logs, filters, tail, grouping)
- `src/stores/connectionStore.ts` - AWS connection state, log groups
- `src/stores/settingsStore.ts` - Persisted settings (colors, patterns, time presets)
- `src/types/workspace.ts` - Layout tree types (LayoutNode, LeafNode, SplitNode, WorkspaceConfig)
- `src/components/EditorGroupView.tsx` - Per-group tab bar, drag-drop, split buttons, merged view
- `src/components/PanelContainer.tsx` - Recursive layout tree renderer with resize handles
- `src/components/LogViewer.tsx` - Virtualized log list
- `src/components/LogGroupSelector.tsx` - Fuzzy search log group selector (Fuse.js + virtualized dropdown)
- `src/components/FilterBar.tsx` - Filter input and level toggles
- `src/components/FindBar.tsx` - Find-in-log search bar (CMD-F)
- `src/components/ContextMenu.tsx` - Right-click context menu for log rows
- `src/components/SettingsDialog.tsx` - Settings dialog (CMD-,)
- `src/components/StatusBar.tsx` - Status bar with log counts and cache usage
- `src/hooks/useFindInLog.ts` - Find-in-log state management hook
- `src/utils/highlightMatches.ts` - Text search and highlight utilities
- `src/utils/groupLogs.ts` - Log grouping logic (by stream and Lambda invocation)
- `src/hooks/useLogGroups.ts` - Hook for computing grouped display items
- `src/components/GroupHeader.tsx` - Group header component for stream/invocation headers
- `src/stores/LiveTailManager.ts` - Stream/poll transport orchestrator for live tail
- `src/stores/TailPoller.ts` - Polling transport (fallback for live tail)
- `src/demo/demoStore.ts` - Demo mode state (Zustand, non-persisted)
- `src/demo/demoInvoke.ts` - Invoke wrapper that intercepts Tauri commands in demo mode
- `src/demo/mockData.ts` - Mock Lambda log groups and log event generators
- `src/demo/DemoTailTransport.ts` - Simulated live tail transport for demo mode
- `src/stores/TailTransport.ts` - Transport interface
- `src/types/index.ts` - TypeScript type definitions
- `src/components/UpdateDialog.tsx` - Auto-update dialog with changelog display and release notes link
- `src/hooks/useUpdateCheck.ts` - Hook for checking updates (startup + manual via menu)
- `src/components/TimePresetEditor.tsx` - Time preset editor for Settings dialog

## Development

```bash
npm start          # Run app in dev mode with hot reload
npm run app:build  # Build production app
npm run fmt        # Format code
npm run lint       # Lint code
npm test           # Run tests
npm run test:watch # Run tests in watch mode
```

## Code Style

- Use TypeScript strict mode
- Prefer functional components with hooks
- Use Zustand for state management
- Follow existing patterns in codebase
- Pre-commit hook runs `fmt`, `lint`, and `build` automatically

## Testing

Uses Vitest with React Testing Library. Tests are in `*.test.ts` / `*.test.tsx` files alongside source.

```bash
npm test           # Run all tests
npm run test:watch # Run in watch mode
```

## Common Tasks

### Adding a new component

1. Create component in `src/components/`
2. Export from component file
3. Import where needed
4. Run `npm run fmt && npm run lint`

### Modifying log parsing

- Edit `parseLogLevel()` in `src/utils/logParsing.ts`
- Default log levels: `error`, `warn`, `info`, `debug`, `trace`, `system`, `unknown`
- Log levels are configurable in Settings (colors, keywords, default visibility)

### Adding Tauri commands

1. Add command in `src-tauri/src/lib.rs`
2. Register in `tauri::Builder`
3. Call from frontend via `invoke()`

## Keyboard Shortcuts

| Shortcut           | Action                                       |
| ------------------ | -------------------------------------------- |
| `⌘F` / `Ctrl+F`    | Find text in logs                            |
| `⌘L` / `Ctrl+L`    | Focus filter input and select all            |
| `⌘R` / `Ctrl+R`    | Refresh - reconnect to AWS and re-query logs |
| `⌘K` / `Ctrl+K`    | Clear logs (keep filters, re-fetch)          |
| `⌘N` / `Ctrl+N`    | Open a new Loggy window (new process)        |
| `⌘,` / `Ctrl+,`    | Open Settings                                |
| `⌘A` / `Ctrl+A`    | Select all visible logs                      |
| `⌘C` / `Ctrl+C`    | Copy selected messages to clipboard          |
| `Tab`              | Focus log viewer for keyboard navigation     |
| `↑` / `↓`          | Navigate between log rows                    |
| `Page Up` / `Down` | Jump one page at a time                      |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aegixx/aws-loggy](https://github.com/aegixx/aws-loggy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
