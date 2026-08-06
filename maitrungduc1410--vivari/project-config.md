---
trigger: always_on
description: handles BOTH requests and responses; do NOT special-case it to responses.
---

# AGENTS.md

Guidance for AI agents (and humans) working in this repo. Read this first, then
read [`ARCHITECTURE.md`](./ARCHITECTURE.md) before touching the runtime, the
protocol, or networking. [`roadmap.md`](./roadmap.md) is the chronological log of
what was built and *why* — search it before assuming something is missing.

---

## What this project is

Vivari is an open-source **WebContainer**: it runs Node-style projects
(Vite + HMR, React, NestJS, Express, `npm install`, `tsc`) **entirely in the
browser tab**, with no backend doing the work. The filesystem, a Node-compatible
runtime, a process/PID model, and TCP-style networking are all emulated across
Web Workers.

The guiding philosophy is **run the real thing**: we vendor Node's actual
`lib/*.js` on a small binding layer, run unmodified npm packages from disk, and
drive real tools (rolldown/Vite, `tsc`, Babel) in-VM. When something breaks, the
fix is almost always "make our emulation match real Node," not "special-case the
tool."

For the full mental model (worker topology, syscall protocol, networking seams,
event loop, native Wasm), read **[`ARCHITECTURE.md`](./ARCHITECTURE.md)**.

---

## Folder structure

```
packages/
  vfs/             Rust → Wasm VFS (inode tree, stat/symlink/rename, errno).
  codec/           Rust → Wasm zlib/deflate core (beneath lib/zlib.js).
  crypto/          Rust → Wasm crypto core (beneath lib/crypto.js).
  wasi-demo/       Rust → wasm32-wasip1 CLI to exercise the WASI layer.

  protocol/
    syscall.js     THE worker↔host ABI over one SharedArrayBuffer. 1 MiB window.
                   Single source of truth for the wire format + opcodes.

  kernel-host/     The supervisor (environment-agnostic).
    kernel.js      PID table, spawn/kill/waitpid, net port registry,
                   HTTP request routing, OP_RESPOND reassembly.
    fs-server.js   FsServer: owns the one VFS, services fs opcodes over each SAB.
    kernel-fs.js   kernel-side sync fs helper.
    coreutils.js   echo/cat/ls/pwd/... + a small `sh`.
    opfs-persistence.js  write-behind mirror of the VFS to OPFS (survives reload).
    node-gyp-stub.js     node-gyp no-op stub (native builds non-fatal) for real npm.
    load-real-npm.js     unpack the vendored real-npm asset into the VFS + shim /bin/npm.
    load-real-tsgo.js     unpack the vendored TypeScript-7 (tsgo, Go/wasm) asset + shim /bin/tsc,/bin/tsgo.
    programs/npm.js       from-scratch npm installer — LEGACY fallback (see real npm below).
    programs/bun.js       Node-backed `bun`/`bunx` shim (always in COREUTILS; not a vendored pack).

  runtime/         The Node runtime that runs INSIDE each process worker.
    index.js       createRuntime(): wires builtins/globals/http-bridge/ws + run().
    module.js      synchronous CommonJS loader (require + resolution).
    toolchain-shims.js  single source of truth for native->wasm drop-ins (NATIVE_WASM_ALIASES).
    esbuild-inproc-patch.js  load-time, version-agnostic rewrite of esbuild-wasm's service to run in-process.
    esm.js         ESM→CJS transpiler (import/export → sync CJS).
    typescript-transform.js  synchronous, dependency-free TS/JSX type-strip + JSX
                   lowering for the loader (Bun's zero-config .ts/.tsx exec; gated so plain JS is untouched).
    loop.js        the per-process event loop (nextTick→micro→timers→immediate).
    boot.js        process bootstrap shared by browser + Node worker entries.
    fs-client.js   env-agnostic Atomics syscall client (the caller side).
    websocket.js   in-VM WebSocket client (used by the HMR tunnel).
    builtins/      hand-written: process, os, assert, child_process, bun (Bun global + bun:* modules).
    node/
      lib/         Node's REAL vendored lib/*.js (fs, net, http, stream, ...).
      internal/    Node's REAL internal/* (streams, errors, validators, ...).
      bindings/    our internalBinding shims (fs, tcp_wrap, zlib, crypto, ...).
        http_parser.js  selects the HTTP parser: real llhttp-in-Wasm (default),
                        pure-JS fallback. Force with VV_HTTP_PARSER=js|wasm.
        llhttp/      llhttp compiled to Wasm (vendored from undici) + the bridge
                     (llhttp-parser.js) folding llhttp callbacks onto Node's
                     HTTPParser contract; regen the binary via scripts/vendor-llhttp.mjs.
      internal-binding.js / primordials.js / loader.js   glue for the above.

  studio/          The primary UI: a Vite + React 19 (React Compiler) + Tailwind v4
                   + shadcn/ui + Iconify app. Vite is the single toolchain and also
                   BUNDLES the worker roles below + the wasm (nested module workers
                   via `new Worker(new URL(...), {type:'module'})`, wasm via
                   `new URL(..._bg.wasm, import.meta.url)`). Run with `npm run dev`.
    vite.config.ts   COOP/COEP headers (dev + preview) + `Service-Worker-Allowed:/`
                     for /sw.js + `worker.format:'es'` + `server.fs.allow` (repo root,
                     so it can read the sibling worker/wasm sources) + React Compiler
                     (plugin-react v6 is oxc-based; the compiler is wired via
                     `reactCompilerPreset()` + `@rolldown/plugin-babel`) + `serveDevtools()`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maitrungduc1410/vivari](https://github.com/maitrungduc1410/vivari) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
