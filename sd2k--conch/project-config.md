---
trigger: always_on
description: Context for AI agents working on this codebase.
---

# Conch — Agent Notes

Context for AI agents working on this codebase.

## Important: Read the Notes

Before making significant changes, check the `notes/` directory:

- `vfs-architecture.md` — How hybrid VFS works via WASI shadowing (eryx pattern)
- `wasip1-vs-wasip2.md` — Why we use wasip2 component model

These documents explain *why* things are the way they are.

## Important: Use mise for all commands

**Always prefer `mise run <task>` over manual `cargo` commands.** The mise tasks handle proper build ordering, feature flags, and dependencies.

```bash
# Good - use mise tasks
mise run build
mise run test
mise run lint

# Avoid - manual cargo commands may miss dependencies
cargo build
```

See "Commands (via mise)" below for the full list.

## What This Project Does

Conch is a **sandboxed shell execution engine**. It compiles a bash-compatible shell (brush) to WebAssembly and runs it via wasmtime with:

- **Hybrid VFS**: Combine in-memory storage with real filesystem mounts
- **Strict resource limits**: CPU time, memory, output size
- **Capability-based security**: Only explicitly mounted paths are accessible

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  Host Application (Rust/Go)                                 │
│                                                             │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  Shell API (high-level)                                │ │
│  │  - ShellBuilder: configure VFS paths and real mounts   │ │
│  │  - Shell::execute(): run commands with hybrid VFS      │ │
│  │  - VfsStorage: in-memory or custom storage backend     │ │
│  └────────────────────────────────────────────────────────┘ │
│                           │                                 │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  Conch API (simple)                                    │ │
│  │  - Conch::execute(): run commands without VFS          │ │
│  │  - Concurrency limiting via semaphore                  │ │
│  └────────────────────────────────────────────────────────┘ │
│                           │                                 │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  ComponentShellExecutor (low-level)                    │ │
│  │  - execute(): basic WASI execution                     │ │
│  │  - execute_with_hybrid_vfs(): VFS + real mounts        │ │
│  │  - InstancePre for fast per-call instantiation         │ │
│  └────────────────────────────────────────────────────────┘ │
│                           │                                 │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  wasmtime runtime                                      │ │
│  │  - WASI Preview 2 (wasip2) component model             │ │
│  │  - eryx-vfs: hybrid VFS via WASI shadowing             │ │
│  │  - Epoch-based CPU interruption                        │ │
│  │  - Memory limits via ResourceLimiter                   │ │
│  └────────────────────────────────────────────────────────┘ │
│                           │                                 │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  conch-shell (WASM component)                          │ │
│  │  - brush-core: full bash interpreter                   │ │
│  │  - Custom builtins: cat, grep, head, tail, jq, wc      │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

## Key Design Decisions

1. **Hybrid VFS via eryx-vfs**: The shell sees a unified filesystem that combines:
   - VFS paths (e.g., `/scratch`) backed by `VfsStorage` trait
   - Real paths (e.g., `/project`) backed by cap-std `Dir` handles
   - WASI filesystem interfaces are shadowed to route to the appropriate backend

2. **wasip2 component model**: We use WASI Preview 2 for better composability and the component model's cleaner interface boundaries.

3. **Three API levels**:
   - `Shell`: High-level builder pattern, recommended for most uses
   - `Conch`: Simple API with concurrency limiting, no VFS
   - `ComponentShellExecutor`: Low-level control over WASM execution

4. **purego over CGO**: Go bindings use purego for CGO-free cross-compilation.

5. **InstancePre for performance**: The executor pre-links the WASM component once, then creates a fresh Store per execution.

## Repository Layout

```
crates/
├── conch/                    # Main host library
│   ├── src/lib.rs           # Public API exports
│   ├── src/shell.rs         # Shell and ShellBuilder (high-level API)
│   ├── src/runtime.rs       # Conch struct, ExecutionResult
│   ├── src/executor/        # WASM executors
│   │   ├── mod.rs           # Module exports
│   │   └── component.rs     # ComponentShellExecutor (wasip2)
│   ├── src/limits.rs        # ResourceLimits struct
│   └── src/ffi.rs           # C FFI exports for Go
│
├── conch-mcp/                # MCP server for AI assistant integration
│   ├── src/lib.rs           # ConchServer with execute tool
│   └── src/main.rs          # MCP server binary (stdio transport)
│
├── conch-shell/              # WASM shell component
│   ├── src/lib.rs           # WIT component entry point

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sd2k/conch](https://github.com/sd2k/conch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
