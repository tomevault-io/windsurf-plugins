---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

HyperDesk is a Windows-only enterprise desktop app that unifies Hyper-V VM management and remote desktop sessions (RDP, VMware Horizon) into a 2×2 grid interface. Its core innovation is **SwallowGrid™** — a Win32 technique that embeds external application windows (mstsc.exe, vmconnect.exe, Horizon client) directly into React grid slots using `SetParent`/`SetWindowPos`.

Tech stack: React 19 + TypeScript + Vite (frontend), Tauri v2 + Rust (backend), Win32 API (window swallowing), PowerShell (Hyper-V automation).

## Commands

```bash
# Development
npm install                 # Install dependencies
npm run tauri dev           # Full dev mode (Rust + React with HMR)
npm run dev                 # Frontend-only (port 1420, no Rust backend)

# Build & Release
npm run build               # TypeScript check + Vite bundle
npm run tauri build         # Production build → NSIS installer (dist/)

# Rust only (from src-tauri/)
cargo build                 # Compile Rust backend
cargo test                  # Run Rust unit tests (hosts.rs has persistence tests)
cargo clippy                # Rust linting
```

Release builds are triggered by pushing a `v*` tag — GitHub Actions builds the MSI on Windows runners via `.github/workflows/release.yml`.

## Architecture

### Frontend → Backend Communication

All IPC goes through `src/lib/tauri-api.ts`, which wraps `invoke<T>()` from `@tauri-apps/api/core`. The file also exports mock fallbacks so the frontend can be tested in a browser without Tauri. When adding new commands, add both the Rust handler in `src-tauri/src/commands.rs` and a typed wrapper in `tauri-api.ts`.

Backend → Frontend async events use `app.emit()` in Rust and `listen<T>()` in the frontend (from `@tauri-apps/api/event`). Key events: `hotkey-focus`, `swallow-success`, `swallow-failure`, `swallow-progress`, `window-closed`.

### Window Swallowing (`src-tauri/src/swallow.rs`)

The core Win32 engine. Flow:
1. Find the target process window by PID using `EnumWindows`
2. Call `SetParent(hwnd, webview_container_hwnd)` to reparent the window into the WebView container (Chrome_WidgetWin)
3. `SetWindowPos` to position/resize within the slot bounds
4. A background stabilization thread polls with adaptive backoff (100ms → 1s once stable, any correction resets to fast) for the **life of the swallow** to enforce styles/position — it doubles as the slot watchdog: its `IsWindow` check detects a crashed/closed child and emits `window-closed` (do not re-add a deadline; a child dying after it left the slot showing a corpse)

**Z-order reality**: After `SetParent`, the swallowed Win32 window sits **above** the WebView2 renderer within Chrome_WidgetWin. This means HTML elements in slot areas are hidden behind the swallowed window. To keep controls accessible, the slot uses a **permanent 36px `slot-header-bar`** at the top — positioned above where the Win32 child starts — so header buttons are always reachable.

**SlotLayout pattern** (`SwallowSlot.tsx`):
- Outer `.swallow-slot`: flex column container (no ref)
- `.slot-header-bar`: 36px, always rendered when `isSwallowed`, never covered by Win32 child
- `.slot-content-area` (ref=`contentRef`): flex:1, this is what `getBoundingClientRect()` measures and what the Win32 window fills

**VMConnect ribbon**: VMConnect has a non-removable 30px client-area ribbon. Fix: position window at `y - 30`, height `+ 30`, then call `SetWindowRgn(CreateRectRgn(0, 30, w, h+30))` to mask the ribbon from view. The stabilization loop reapplies the region if VMConnect resets it.

**RDP settings** (in `commands.rs` `connect_vm`): uses `screen mode id:i:1` (windowed — the connection bar is a fullscreen-only element, so it never exists), `smart sizing:i:1` (classic mstsc can't renegotiate resolution mid-session; the bitmap is scaled to the slot instead — connect happens at full primary-monitor resolution to minimize blur), `keyboardhook:i:1` (Win key/Alt+Tab go to the remote whenever the session has focus; Alt+1~4 slot switching survives because the LL keyboard hook intercepts those before the remote sees them). `authentication level:i:2` (warn-but-allow) — **never set this to `i:0`**; it silently bypasses server identity verification and exposes RDP sessions to MITM.

**Focus forwarding**: `swallow::focus_window(slot_id)` calls `SetForegroundWindow` + `BringWindowToTop`. Called directly from Alt+1–4 hotkey handlers in `lib.rs` and via `focus_slot_window` Tauri command (triggered by `MultiView.tsx` on hotkey events).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qetqet910/HyperDesk](https://github.com/qetqet910/HyperDesk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
