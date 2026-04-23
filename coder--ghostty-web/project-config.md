---
trigger: always_on
description: **For AI coding agents working on this repository.**
---

# Agent Guide - Ghostty WASM Terminal

**For AI coding agents working on this repository.**

## Quick Start

```bash
bun install                          # Install dependencies
bun test                            # Run test suite (95 tests)
bun run dev                         # Start Vite dev server (http://localhost:8000)
```

**Before committing, always run:**

```bash
bun run fmt && bun run lint && bun run typecheck && bun test && bun run build
```

**Run interactive terminal demo:**

```bash
cd demo/server && bun install && bun run start  # Terminal 1: PTY server
bun run dev                                     # Terminal 2: Web server
# Open: http://localhost:8000/demo/
```

## Project State

This is a **fully functional terminal emulator** (MVP complete) that uses Ghostty's battle-tested VT100 parser compiled to WebAssembly.

**What works:**

- ✅ Full VT100/ANSI terminal emulation (vim, htop, colors, etc.)
- ✅ Canvas-based renderer with 60 FPS
- ✅ Keyboard input handling (Kitty keyboard protocol)
- ✅ Text selection and clipboard
- ✅ WebSocket PTY integration (real shell sessions)
- ✅ xterm.js-compatible API
- ✅ FitAddon for responsive sizing
- ✅ Comprehensive test suite (terminal, renderer, input, selection)

**Tech stack:**

- TypeScript + Bun runtime for tests
- Vite for dev server and bundling
- Ghostty WASM (404 KB, committed) for VT100 parsing
- Canvas API for rendering

## Architecture

```
┌─────────────────────────────────────────┐
│  Terminal (lib/terminal.ts)             │  xterm.js-compatible API
│  - Public API, event handling           │
└───────────┬─────────────────────────────┘
            │
            ├─► GhosttyTerminal (WASM)
            │   └─ VT100 state machine, screen buffer
            │
            ├─► CanvasRenderer (lib/renderer.ts)
            │   └─ 60 FPS rendering, all colors/styles
            │
            ├─► InputHandler (lib/input-handler.ts)
            │   └─ Keyboard events → escape sequences
            │
            └─► SelectionManager (lib/selection-manager.ts)
                └─ Text selection + clipboard

Ghostty WASM Bridge (lib/ghostty.ts)
├─ Ghostty - WASM loader
├─ GhosttyTerminal - Terminal instance wrapper
└─ KeyEncoder - Keyboard event encoding
```

### Key Files

| File                        | Lines | Purpose                             |
| --------------------------- | ----- | ----------------------------------- |
| `lib/terminal.ts`           | 427   | Main Terminal class, xterm.js API   |
| `lib/ghostty.ts`            | 552   | WASM bridge, memory management      |
| `lib/renderer.ts`           | 610   | Canvas renderer with font metrics   |
| `lib/input-handler.ts`      | 438   | Keyboard → escape sequences         |
| `lib/selection-manager.ts`  | 442   | Text selection + clipboard          |
| `lib/types.ts`              | 454   | TypeScript definitions for WASM ABI |
| `lib/addons/fit.ts`         | 240   | Responsive terminal sizing          |
| `demo/server/pty-server.ts` | 284   | WebSocket PTY server (real shell)   |

### WASM Integration Pattern

**What's in Ghostty WASM:**

- VT100/ANSI state machine (the hard part)
- Screen buffer (2D cell grid)
- Cursor tracking
- Scrollback buffer
- SGR parsing (colors/styles)
- Key encoding

**What's in TypeScript:**

- Terminal API (xterm.js compatibility)
- Canvas rendering
- Input event handling
- Selection/clipboard
- Addons (FitAddon)
- WebSocket/PTY integration

**Memory Management:**

- WASM exports linear memory
- TypeScript reads cell data via typed arrays
- No manual malloc/free needed (Ghostty manages internally)
- Get cell pointer: `wasmTerm.getScreenCells()`
- Read cells: `new Uint8Array(memory.buffer, ptr, size)`

## Development Workflows

### Before Committing

**⚠️ Always run all CI checks before committing:**

```bash
bun run fmt                           # Check formatting (Prettier)
bun run lint                          # Run linter (Biome)
bun run typecheck                     # Type check (TypeScript)
bun test                              # Run tests (95 tests)
bun run build                         # Build library
```

All at once: `bun run fmt && bun run lint && bun run typecheck && bun test && bun run build`

Auto-fix formatting: `bun run fmt:fix`

### Running Tests

```bash
bun test                              # Run all tests
bun test lib/terminal.test.ts         # Run specific file
bun test --watch                      # Watch mode (may hang - use Ctrl+C and restart)
bun test -t "test name pattern"       # Run matching tests
```

**Test files:** `*.test.ts` in `lib/` (terminal, renderer, input-handler, selection-manager, fit)

### Running Demos

**⚠️ CRITICAL: Use Vite dev server!** Plain HTTP server won't handle TypeScript imports.

```bash
# ✅ CORRECT
bun run dev                           # Vite with TS support
# Open: http://localhost:8000/demo/

# ❌ WRONG
python3 -m http.server                # Can't handle .ts imports
```

**Available demos:**

- `demo/index.html` - Interactive shell terminal (requires PTY server)
- `demo/colors-demo.html` - ANSI color showcase (no server needed)

### Type Checking

```bash
bun run typecheck                     # Check types without compiling
```

### Debugging

**Browser console (F12):**

```javascript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coder/ghostty-web](https://github.com/coder/ghostty-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
