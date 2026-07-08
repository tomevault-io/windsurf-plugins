---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

Flashot is a fast, cross-platform screenshot tool built with Tauri 2 + React + TypeScript. It captures screens via global hotkey, displays an overlay for region selection, and copies or saves the result.

**Key architectural pattern**: The app uses a **session-based capture model** with RAII guards. When the hotkey fires, Rust captures all monitors into frozen frames, spawns overlay windows (one per monitor), and holds a `SessionGuard`. The guard's drop automatically cleans up overlays and frames, ensuring no resource leaks even on error paths.

## Development Commands

### Frontend

```bash
pnpm dev              # Run Vite dev server (frontend only)
pnpm build            # Build frontend for production
pnpm test             # Run Vitest tests
pnpm test:watch       # Run tests in watch mode
pnpm lint             # TypeScript type checking
```

### Rust Backend

```bash
cd src-tauri
cargo check           # Fast compile check
cargo clippy          # Linting (must pass with -D warnings)
cargo test            # Run unit tests
cargo bench           # Run all benchmarks
cargo bench --bench crop_bench  # Run specific benchmark
cargo build --release # Production build
```

### Full App

```bash
pnpm tauri dev        # Run full app in dev mode
pnpm tauri build      # Build production bundle (.dmg, .msi, .AppImage)
```

### Local CI Preflight

```bash
pnpm ci:local         # Run frontend checks, Rust checks, crop bench, and Linux/Windows smoke checks
```

Run `pnpm ci:local` before pushing changes, especially after dependency updates or edits under `src-tauri/src/capture/`, `src-tauri/src/window_probe/`, platform `cfg(...)` blocks, or Cargo manifests. It includes smoke checks for Linux-only `ashpd` screenshot APIs and Windows-only `windows` crate APIs, catching platform-gated compile errors that macOS-only `cargo check` misses.

## Architecture

### Capture Flow (Rust → Frontend → Rust)

1. **Hotkey trigger** (`src-tauri/src/lib.rs:run_capture`)
   - Captures all monitors in parallel with `xcap`
   - Enumerates windows with platform-specific APIs
   - Saves frames as PNGs in app cache dir
   - Emits `capture:start` event to each overlay window with frame URL + window rects

2. **Overlay interaction** (`src/routes/Overlay.tsx` + `src/overlay/state.ts`)
   - Zustand store manages state machine: `idle → hover → dragging → committed`
   - Mouse events drive state transitions
   - Window detection uses z-order hit-testing (`src/lib/hit-test.ts`)
   - Selection handles use geometry utilities (`src/lib/geometry.ts`)

3. **Crop & output** (`src-tauri/src/commands.rs`)
   - Frontend calls `cropAndCopy` or `cropAndSave` with monitor ID + rect
   - Rust retrieves frozen frame from `WindowMgr`, crops with scale factor, outputs to clipboard/file
   - `SessionGuard` drop cleans up overlays and frames

### Key Rust Modules

- **`window_mgr.rs`**: Session lifecycle manager. `SessionGuard` is RAII — drop always calls `end()` to hide overlays and clear frames. Never manually manage session state.
- **`capture/`**: Platform-specific screen capture (macOS uses `xcap`, Windows uses `xcap` + Win32 APIs)
- **`window_probe/`**: Platform-specific window enumeration (macOS: Core Graphics, Windows: Win32)
- **`hotkey.rs`**: Global hotkey registration with live updates on settings change
- **`commands.rs`**: Tauri command handlers. All commands receive `State<Arc<WindowMgr>>` to access frozen frames.

### Key Frontend Modules

- **`src/overlay/state.ts`**: Zustand store for overlay state machine. All overlay components read from this store.
- **`src/lib/geometry.ts`**: Pure functions for rect operations (clamp, resize, translate). Used by selection handles.
- **`src/lib/hit-test.ts`**: Z-order window hit-testing. Returns topmost window at cursor position.
- **`src/lib/ipc.ts`**: Typed wrappers around Tauri IPC (commands + events). Use these instead of raw `invoke()`.

### Multi-Monitor Handling

Each monitor gets its own overlay window (label: `overlay-{monitor_id}`). The overlay route listens for `capture:start` events, which include:

- `monitorId`: Which monitor this overlay belongs to
- `frameUrl`: `asset://` URL to the frozen screenshot PNG
- `windows`: Array of window rects translated to monitor-local coordinates

When the user selects a region, the frontend sends the monitor ID + rect to Rust. Rust looks up the frozen frame by monitor ID and crops it.

### Settings Persistence

Settings are stored via `tauri-plugin-store` in JSON format. When settings change:

1. Frontend calls `setSettings` command
2. Rust saves to disk and emits `settings:changed` event
3. Hotkey service listens for this event and re-registers the hotkey

This allows live hotkey updates without app restart.

## Testing

### Frontend Tests

- Located in `src/__tests__/`
- Use Vitest + React Testing Library
- Focus on pure logic (geometry, hit-testing)
- Run with `pnpm test`

### Rust Tests

- Unit tests inline with modules (e.g., `window_mgr.rs` has `#[cfg(test)] mod tests`)
- Run with `cd src-tauri && cargo test`

### Benchmarks

- Located in `src-tauri/benches/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [poneding/flashot](https://github.com/poneding/flashot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
