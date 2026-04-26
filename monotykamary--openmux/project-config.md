---
trigger: always_on
description: This file provides guidance to coding agents working with the openmux repository.
---

# AGENTS.md

This file provides guidance to coding agents working with the openmux repository.

## Build and Run Commands

```bash
bun install           # Install dependencies
bun start             # Build and run the terminal multiplexer
bun dev               # Run with watch mode (--watch)
bun run typecheck     # Type check without emitting
bun run lint          # Lint Effect code (effect-language-service)
bun run build         # Build standalone binary (./scripts/build.sh)
bun run build:release # Build optimized binary
bun run install:local # Build and install locally
bun run test          # Run bun:test TS + Zig + Ghostty VT tests
bun run test:ts       # Run bun:test suite only
bun run test:pty      # Run Zig PTY tests only
bun run test:ghostty-vt # Run Ghostty VT Zig tests
bun run test:watch    # Run bun:test in watch mode
bun run check:circular # Detect circular deps in src/
```

## libghostty-vt Notes

- Ghostty is tracked as a submodule in `vendor/ghostty`.
- Wrapper library lives in `native/zig-ghostty-wrapper` and exports the C API.
- `scripts/update-ghostty-vt.sh` updates the submodule only (no patches).
- `scripts/update-libgit2.sh` updates `vendor/libgit2` directly (no patches).
- `scripts/build.sh` builds `libghostty-vt` via the wrapper before bundling.

## Technology Stack

- **Bun** - Runtime, package manager, and test runner (`bun:test`)
- **OpenTUI** - Terminal UI library with SolidJS reconciler (@opentui/core, @opentui/solid)
- **SolidJS** - Reactive UI framework via OpenTUI's SolidJS reconciler
- **zig-pty** - PTY support for shell process management (pure Zig implementation)
- **libghostty-vt** - Native terminal emulator (VT parsing/state)
- **errore** - Type-safe error handling with Golang-style returns (replaces heavy Effect.ts)

## Architecture Overview

openmux is a terminal multiplexer with a master-stack tiling layout (Zellij-style). The UI is SolidJS components rendered to the terminal via OpenTUI, with PTYs managed in Zig and emulated via native libghostty-vt.

### Entry Points

- `src/index.tsx` - CLI entry, renderer setup
- `src/shim/main.ts` - Shim server entry (background detach/attach)
- `src/App.tsx` - Provider tree and top-level app wiring

### Core Data Flow

```
Keyboard Input → KeyboardContext → Layout/Terminal actions
                                ↓
                    Master-stack layout calculation
                                ↓
                       PaneContainer/TerminalView
PTY Data → zig-pty → GhosttyVTEmulator (libghostty-vt) → Shim protocol → TerminalContext
                                ↓
                   TerminalView + AggregateView preview
Session data → SessionContext → disk persistence → SessionBridge (Layout/Terminal)
Title updates → TitleContext → StatusBar/window title
```

Note: In shim server mode, PTY data stays local; in shim client mode, it flows through the shim protocol.

Detach/attach uses a single-client lock; new clients steal the lock and detach the previous client.

### Context Hierarchy (src/App.tsx)

```tsx
ThemeProvider              // Styling/theming
  └── LayoutProvider       // Workspace/pane state (reducer pattern)
        └── KeyboardProvider    // Prefix mode, key state
              └── TitleProvider     // Terminal title updates
                    └── TerminalProvider   // PTY lifecycle, terminal state
                          └── SelectionProvider  // Text selection state
                                └── SearchProvider    // Terminal search
                                      └── SessionBridge   // SessionProvider wiring
                                            └── AggregateViewProvider  // Cross-workspace overlay
                                                  └── AppContent
```

### Key Modules

**Layout and session state (src/core/)**
- `types.ts`, `config.ts` - Core types and defaults
- `operations/master-stack-layout.ts` - Layout calculation
- `operations/layout-actions/` - Pane/workspace actions
- `operations/session-actions/` - Session restore/save helpers
- `workspace-utils.ts`, `coordinates/`, `scroll-utils.ts`, `keyboard-utils.ts`

**Terminal layer (src/terminal/)**
- `ghostty-vt/`, `emulator-utils/` - native libghostty-vt bindings + shared emulator helpers
- `emulator-interface.ts` - ITerminalEmulator abstraction
- `input-handler.ts`, `sync-mode-parser.ts` - Input/escape handling
- `title-parser.ts`, `terminal-query-passthrough/` - Title/query parsing
- `paste-intercepting-stdin.ts`, `focused-pty-registry.ts`

**Shim / detach (src/shim/)**
- `main.ts`, `server.ts` - Shim server + RPC handling
- `client/` - Shim client connection, PTY state cache, detach handling

**UI components (src/components/)**
- `PaneContainer.tsx`, `Pane.tsx`, `TerminalView.tsx` - Pane rendering
- `AggregateView.tsx`, `SessionPicker.tsx`, `SearchOverlay.tsx`
- `StatusBar.tsx`, `CopyNotification.tsx`

**SolidJS contexts (src/contexts/)**
- `LayoutContext.tsx`, `TerminalContext.tsx`, `KeyboardContext.tsx`
- `SelectionContext.tsx`, `SearchContext.tsx`, `SessionContext.tsx`
- `ThemeContext.tsx`, `TitleContext.tsx`, `AggregateViewContext.tsx`

**Effect module (src/effect/)**
- `errors.ts` - Domain-specific tagged errors using errore
- `resources.ts` - ResourceStack for cleanup with `using`/`defer`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [monotykamary/openmux](https://github.com/monotykamary/openmux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
