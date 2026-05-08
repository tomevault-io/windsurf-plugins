---
trigger: always_on
description: Generic environment runner for Node.js. Ported from the nitro env runner concept into a standalone package.
---

# env-runner

Generic environment runner for Node.js. Ported from the nitro env runner concept into a standalone package.

> **Note:** Keep `AGENTS.md` updated with project status and structure.

> **Note:** Keep `README.md` usage section updated when adding/changing public API, CLI flags, or runner behavior.

## Architecture

```
src/
├── common/
│   ├── base-runner.ts       # BaseEnvRunner abstract class
│   └── worker-utils.ts      # AppEntry interface, resolveEntry(), parseServerAddress()
├── runners/
│   ├── node-worker/
│   │   ├── runner.ts        # NodeWorkerEnvRunner
│   │   └── worker.ts        # Built-in srvx worker (parentPort)
│   ├── node-process/
│   │   ├── runner.ts        # NodeProcessEnvRunner
│   │   └── worker.ts        # Built-in srvx worker (process.send)
│   ├── bun-process/
│   │   ├── runner.ts        # BunProcessEnvRunner
│   │   └── worker.ts        # Built-in srvx worker (Bun/Node.js)
│   ├── deno-process/
│   │   ├── runner.ts        # DenoProcessEnvRunner
│   │   └── worker.ts        # Built-in srvx worker (Deno)
│   ├── self/
│   │   └── runner.ts        # SelfEnvRunner (in-process, no worker)
│   ├── miniflare/
│   │   └── runner.ts        # MiniflareEnvRunner (Cloudflare Workers via miniflare)
│   ├── vercel/
│   │   ├── runner.ts        # VercelEnvRunner (extends NodeWorkerEnvRunner)
│   │   └── worker.ts        # Sets Vercel request context symbol, delegates to node-worker
│   └── netlify/
│       ├── runner.ts        # NetlifyEnvRunner (extends NodeWorkerEnvRunner)
│       └── worker.ts        # Sets global Netlify context, delegates to node-worker
├── types.ts                 # Core interfaces
├── index.ts                 # Public API exports
├── loader.ts                # Dynamic runner loader
├── manager.ts               # RunnerManager for hot-reload
├── server.ts                # EnvServer (high-level API with watch mode)
└── cli.ts                   # CLI entry point
```

- **`src/vite.ts`** — Vite Environment API helpers: `createViteHotChannel()` (host-side HotChannel from runner RPC hooks) and `createViteTransport()` (worker-side ModuleRunner transport)
- **`src/types.ts`** — Core interfaces: `EnvRunner`, `WorkerAddress`, `WorkerHooks`, `RunnerRPCHooks`, `RPCOptions`
- **`src/common/base-runner.ts`** — `BaseEnvRunner` abstract class + `EnvRunnerData`: shared logic for all runners (fetch proxy with exponential backoff, upgrade, message dispatch, socket cleanup)
- **`src/common/worker-utils.ts`** — Shared utilities for built-in workers: `AppEntry` interface (with optional `websocket`, `upgrade`, and `ipc` hooks), `AppEntryIPC`/`AppEntryIPCContext` types, `resolveEntry()` to dynamically import user entry, `parseServerAddress()` to extract host/port from srvx server, `reloadEntryModule()` for cache-busted re-import with IPC teardown/re-init
- **`src/runners/node-worker/runner.ts`** — `NodeWorkerEnvRunner` extends `BaseEnvRunner`: spawns Node.js Worker threads, data via `workerData`
- **`src/runners/node-worker/worker.ts`** — Built-in srvx worker: reads `data.entry` from `workerData`, starts srvx server, reports address via `parentPort`
- **`src/runners/node-process/runner.ts`** — `NodeProcessEnvRunner` extends `BaseEnvRunner`: spawns a child process via `fork()`, supports custom `execArgv`
- **`src/runners/node-process/worker.ts`** — Built-in srvx worker: reads `data.entry` from `ENV_RUNNER_DATA`, starts srvx server, reports address via `process.send()`
- **`src/runners/bun-process/runner.ts`** — `BunProcessEnvRunner` extends `BaseEnvRunner`: uses `Bun.spawn()` with IPC when under Bun, falls back to Node.js `fork()` otherwise
- **`src/runners/bun-process/worker.ts`** — Built-in srvx worker: same as node-process worker (works on both Bun and Node.js)
- **`src/runners/deno-process/runner.ts`** — `DenoProcessEnvRunner` extends `BaseEnvRunner`: spawns a `deno run --allow-all` child process with IPC via Node.js `spawn()`. Data passed via `ENV_RUNNER_DATA` env var (JSON). Supports custom `execArgv`
- **`src/runners/deno-process/worker.ts`** — Built-in srvx worker: same as node-process worker (works on Deno via Node.js compat)
- **`src/runners/self/runner.ts`** — `SelfEnvRunner` extends `BaseEnvRunner`: runs entry code in the same process using an in-memory channel registry on `process.__envRunners`
- **`src/runners/miniflare/runner.ts`** — `MiniflareEnvRunner` extends `BaseEnvRunner`: runs entry in Cloudflare Workers runtime via miniflare. Overrides `fetch()` to use `mf.dispatchFetch()`. Uses in-memory `script` (no temp files), `unsafeModuleFallbackService` for module resolution, and `unsafeEvalBinding` for hot-reload via `reloadModule()`. Requires `miniflare` peer dependency
- **`src/runners/vercel/runner.ts`** — `VercelEnvRunner` extends `NodeWorkerEnvRunner`: simulates Vercel deployment environment with header injection
- **`src/runners/vercel/worker.ts`** — Sets `Symbol.for("@vercel/request-context")` on globalThis, delegates to node-worker worker

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unjs/env-runner](https://github.com/unjs/env-runner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
