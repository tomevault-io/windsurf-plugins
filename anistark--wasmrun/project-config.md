---
trigger: always_on
description: > Instructions for Claude Code, pi, Cursor, Copilot, and other AI coding agents working on this project.
---

# AGENTS.md — AI Coding Agent Instructions for Wasmrun

> Instructions for Claude Code, pi, Cursor, Copilot, and other AI coding agents working on this project.

---

## Project Overview

**Wasmrun** is a WebAssembly runtime. It compiles, runs, inspects, and manages WASM modules with multi-language support (Rust, Go, Python, C/C++, AssemblyScript) via a plugin architecture.

- **Repository:** https://github.com/anistark/wasmrun
- **Crate:** https://crates.io/crates/wasmrun
- **Docs:** https://wasmrun.readthedocs.io
- **License:** MIT
- **Minimum Rust Version:** 1.85
- **Recommended Rust Version:** 1.88

---

## ⚠️ The Three Modes — Read This First

Wasmrun has **three distinct execution modes**. They are separate systems with separate philosophies. **Do not conflate them.** When working on one mode, do not break another mode's functionality.

### 1. Server Mode (`wasmrun` / `wasmrun run`)

**Philosophy:** A development server that compiles source code to WASM and serves it in a browser with a UI.

- **Trigger:** `wasmrun run ./project` or just `wasmrun ./project`
- **What it does:** Detects project language → compiles to WASM via plugins → starts HTTP server → serves browser UI that loads and runs the WASM
- **Key files:**
  - `src/commands/run.rs` — command handler
  - `src/config/server.rs` — server config, `run_server()`
  - `src/server/` — HTTP server infrastructure (handler, API, wasm serving, lifecycle)
  - `src/compiler/` — project compilation
  - `src/plugin/` — plugin system (compile plugins)
  - `src/watcher.rs` — live reload file watching
  - `src/template.rs` — HTML template injection
  - `ui/src/` — Preact UI source (builds into `templates/app/`, `templates/console/` at compile time via `build.rs`)
- **Uses plugins:** Yes — plugins provide compilation (wasmrust, wasmgo, waspy, wasmasc)
- **Uses browser:** Yes — serves HTML + JS that loads WASM via `WebAssembly.instantiate()`
- **Docs:** `docs/docs/server/`

### 2. Exec Mode (`wasmrun exec`)

**Philosophy:** A native WASM interpreter. No browser, no server, no compilation. Just parse and execute a `.wasm` binary directly.

- **Trigger:** `wasmrun exec ./file.wasm [args...]`
- **What it does:** Parses WASM binary → initializes memory → links WASI host functions → interprets bytecode → prints output → returns exit code
- **Key files:**
  - `src/commands/exec.rs` — command handler
  - `src/runtime/core/` — **the entire WASM interpreter engine**
    - `module.rs` — binary parser
    - `executor.rs` — instruction executor (~4400 lines, all WASM opcodes)
    - `memory.rs` — linear memory (pages, bounds checking)
    - `values.rs` — value types (i32, i64, f32, f64)
    - `linker.rs` — host function imports/exports linking
    - `native_executor.rs` — high-level API: `execute_wasm_file()`, `execute_wasm_file_with_args()`
    - `control_flow.rs` — control flow analysis
  - `src/runtime/wasi/` — WASI syscall implementations (fd_write, args_get, clock, etc.)
    - `mod.rs` — WasiEnv, create_wasi_linker()
    - `syscalls.rs` — individual syscall host functions
- **Uses plugins:** No
- **Uses browser:** No
- **Docs:** `docs/docs/exec/`

### 3. OS Mode (`wasmrun os`)

**Philosophy:** A browser-based micro-kernel environment. Runs projects (Node.js, Python) inside a WASM VM in the browser with a full development UI (console, filesystem, logs, kernel status).

- **Trigger:** `wasmrun os ./project`
- **What it does:** Detects language → starts HTTP server → serves Preact UI → fetches language runtime WASM from wasmhub → populates virtual FS with project files → boots WASM VM in browser → runs user code sandboxed
- **Key files:**
  - `src/commands/os.rs` — command handler
  - `src/runtime/os_server.rs` — OS mode HTTP server (serves UI, APIs for kernel/fs/logs/tunnel)
  - `src/runtime/multilang_kernel.rs` — multi-language kernel (process management, language detection)
  - `src/runtime/microkernel.rs` — base micro-kernel (process table, WASI, VFS)
  - `src/runtime/dev_server.rs` — per-process dev server (serves WASI filesystem files)
  - `src/runtime/scheduler.rs` — process scheduler
  - `src/runtime/network_namespace.rs` — network isolation, port forwarding
  - `src/runtime/wasi_fs.rs` — virtual filesystem (in-memory, mount points)
  - `src/runtime/project_files.rs` — project file collection for browser transfer
  - `src/runtime/runtime_cache.rs` — language runtime WASM caching (from wasmhub)
  - `src/runtime/tunnel/` — bore tunneling for public access
  - `src/runtime/languages/` — language runtime traits (Node.js, Go, Python)
  - `src/logging/` — structured log trail system
  - `ui/src/` — Preact UI source (components, OS panels, WASI shim; builds into `templates/os/` at compile time via `build.rs`)
- **Uses plugins:** No (uses its own language detection and wasmhub runtimes)
- **Uses browser:** Yes — full Preact UI with console, filesystem, kernel panels
- **Docs:** `docs/docs/os/`

### Mode Boundaries — Critical Rules

1. **Never mix mode-specific logic.** Exec mode must never start an HTTP server. Server mode must never invoke the bytecode interpreter. OS mode has its own kernel — don't route it through the server mode pipeline.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anistark/wasmrun](https://github.com/anistark/wasmrun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
