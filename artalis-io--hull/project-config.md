---
trigger: always_on
description: Hull is a capability-secure, local-first runtime for programmable tools and workflows. It provides structured JSON interfaces for AI coding agents, but also works for human developers and automated services. This document covers everything an agent needs to build, test, debug, and deploy Hull applications.
---

# Hull — Agent Development Guide

Hull is a capability-secure, local-first runtime for programmable tools and workflows. It provides structured JSON interfaces for AI coding agents, but also works for human developers and automated services. This document covers everything an agent needs to build, test, debug, and deploy Hull applications.

## Quick Start

```bash
# Create a new project
hull new myapp && cd myapp

# Start development server (hot-reload)
hull dev --agent app.lua -p 3000

# In another terminal — introspect the app
hull agent routes .                    # list all routes as JSON
hull agent db schema .                 # show database tables and columns
hull agent request GET /health         # HTTP request to the running server
hull agent status .                    # check if dev server is running
hull agent test .                      # run tests with JSON output
```

## Architecture Overview

```
Application Code (Lua or JS)
        ↓
    app.manifest({...})       # declare capabilities
    app.get("/path", fn)      # register routes
    app.use("*", "/*", mw)    # register middleware
        ↓
Capability Layer (C)          # enforces security boundaries
        ↓
    db.query() / db.exec()    # SQLite (WAL mode, parameterized, _hull_* tables blocked)
    fs.read() / fs.mmap()     # sandboxed filesystem (mmap → GPU zero-copy)
    crypto.sha256() / etc.    # cryptographic primitives
    http.get() / http.post()  # outbound HTTP (host allowlist)
    gpu.dispatch() / pipeline # GPU compute (wgpu-native, optional)
        ↓
Keel HTTP Server (C)          # epoll/kqueue event loop + async + thread pool
        ↓
Kernel Sandbox                # pledge+unveil (Linux), C-level (macOS)
```

WASM compute plugins provide a sandboxed data-plane layer for CPU-intensive computation. Plugins are pure functions (no I/O) that run in isolated WASM linear memory with gas metering. Place `.wasm` files in `compute/`, call via `compute.call("name", input)` (sync) or `compute.async.call("name", input)` (async, yields to event loop) from Lua/JS. `compute.stream("name", input, output, opts)` processes data larger than memory in chunks via a persistent instance. `hull build` auto-compiles to AOT when `wamrc` is available (~1.2x native speed vs ~54x for fast interpreter). See `docs/wamr_architecture.md`.

WASM modules can also be registered as SQL UDFs via `db.udf.register("hull_name", "module_name", opts)` — the WASM function is called per row during query execution with gas metering.

GPU compute shaders (optional, `HL_ENABLE_GPU=1`) provide massively parallel data processing via wgpu-native (Metal/Vulkan/DX12). Compile shaders inline with `gpu.compile("name", wgsl)` or load from files with `gpu.load("name")` (reads `shaders/<name>.wgsl`). Dispatch with `gpu.dispatch("name", opts)` or chain multiple shaders with `gpu.pipeline(stages, opts)` for single-submission execution. Persistent buffers (`gpu.buffer()`) keep data on GPU across requests. Persistent textures (`gpu.texture(name, img)`) accept HlImage objects and can be read back via `gpu.texture_read(name)`. Dispatch supports `textures` array for sampled and storage texture bindings. Fire-and-forget mode (`output = false`) updates GPU buffers in-place without readback. `gpu.buffer_copy()` copies between GPU buffers without CPU roundtrip. GPU dispatches time out after 5 seconds (`HL_GPU_TIMEOUT_MS`).

Both WASM and GPU compute accept the same input types via the unified buffer protocol: strings, `MappedBuffer` (from `fs.mmap()`), and `WasmBuffer` (from `compute.call({buffer=true})`). This enables zero-copy data flow between disk, WASM, and GPU without Lua/JS string intermediaries. Declare `gpu: true` and/or `compute: true` in manifest.

Each app is a single file (`app.lua` or `app.js`) with optional:
- `migrations/*.sql` — database schema (auto-run on startup)
- `templates/*.html` — server-side templates
- `static/*` — served at `/static/*`
- `compute/*.wasm` — WASM compute plugins (auto-AOT compiled during `hull build`)
- `shaders/*.wgsl` — GPU compute shaders (embedded by `hull build`, loaded via `gpu.load()`)
- `tests/test_*.lua` or `tests/test_*.js` — test files

## Runtime Selection

Hull supports two runtimes — selected by file extension:

| Runtime | Extension | Naming | Module Import |
|---------|-----------|--------|---------------|
| Lua 5.4 | `.lua` | `snake_case` | `require("hull.module")` |
| QuickJS (ES2023) | `.js` | `camelCase` | `import { mod } from "hull:module"` |

Both runtimes have identical capabilities. Choose based on preference.

## Agent CLI Commands

All `hull agent` commands output JSON to stdout. Errors go to stderr. Exit code 0 = success, non-zero = error.

### `hull agent routes [app_dir]`

List all registered routes and middleware.

```json
{
  "runtime": "lua",
  "routes": [
    {"method": "GET", "pattern": "/health"},
    {"method": "POST", "pattern": "/tasks"},
    {"method": "GET", "pattern": "/tasks/:id"},
    {"method": "PUT", "pattern": "/tasks/:id"},

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [artalis-io/hull](https://github.com/artalis-io/hull) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
