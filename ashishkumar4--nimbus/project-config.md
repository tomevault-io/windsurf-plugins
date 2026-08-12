---
trigger: always_on
description: Last refreshed: 2026-06-06
---

# AGENTS.md - Nimbus Project Context

Last refreshed: 2026-06-06

Treat live code as the source of truth. Older notes, historical comments, and
generated artifacts may lag; verify against the actual implementation before
making claims.

## Current Shape

Nimbus is a Cloudflare Workers + Durable Objects development environment.
Each browser session maps to a SQLite-backed Durable Object with persistent VFS,
shell state, process tables, port routing, npm/git/runtime substrates, and
hibernation/rehydration support.

The repo is a Bun workspace monorepo:

| Path | What |
|---|---|
| `apps/hosted-demo/` | Live demo / canonical embedder. Deployed at `https://nimbus-os.dev`. |
| `packages/worker/` | `@nimbus-sh/worker`: runtime, `NimbusSession` DO, router, VFS, runtimes, facets, static assets. |
| `packages/sdk/` | `@nimbus-sh/sdk`: Worker embedder exports, token/session helpers, and programmatic sandbox SDK. |
| `packages/react/` | `@nimbus-sh/react`: iframe wrapper component and headless hook. |
| `packages/cli/` | `@nimbus-sh/cli`: scaffold/setup/token/session/runtime commands. |
| `packages/create-nimbus-app/` | `npx create-nimbus-app` wrapper. |
| `packages/config/` | `@nimbus-sh/config`: typed Nimbus and Wrangler config helpers. |
| `tests/behavioral/` | Black-box behavioral probes. |

`apps/hosted-demo/src/index.ts` imports the Worker package through the SDK
entrypoints, exports the required DO/RPC classes, calls `createNimbusHandler`,
and exposes live SDK smoke routes.

## SDK

The programmatic sandbox SDK is implemented in `packages/sdk/src/sandbox.ts`.
It supports:

- `Nimbus.fromEnv(env, config?)` for colocated Workers/DOs with the
  `NIMBUS_SESSION` binding.
- `Nimbus.connect({ endpoint, token, config })` for authenticated remote use.
- `nimbus.sandbox(id, options?)` returning a `NimbusSandbox` with
  `ready`, `exec`, `runCode`, `startProcess`, `files`, `runtimes`,
  `processes`, `ports`, `capabilities`, and `tools()`.
- `@nimbus-sh/sdk/flue` for mapping a Nimbus sandbox to Flue's sandbox
  provider contract without making Flue a hard dependency of the core SDK.

`NimbusSession` exposes the backing RPC methods in
`packages/worker/src/session/nimbus-session.ts`; implementation helpers live in
`packages/worker/src/session/programmatic.ts`.

The SDK is the intended surface for backend sandbox integrations. The hosted
demo also exercises it through `/api/sdk-smoke` and `/api/sdk-remote-smoke`.

## Runtime Internals

Core files:

| File | What |
|---|---|
| `packages/worker/src/session/nimbus-session.ts` | DO class, lifecycle, VFS/facet/process/port ownership, diagnostics, hibernation. |
| `packages/worker/src/session/routes.ts` | DO-internal HTTP/WS routes. |
| `packages/worker/src/session/init.ts` | Session boot, shell commands, npm/git/vite/wrangler/runtime registration. |
| `packages/worker/src/session/agent.ts` | Session Agent API, Cloudflare OAuth flow, AI SDK model calls, sandbox tools. |
| `packages/worker/src/runtime/node-shims.ts` | Node-compatible fs/path/process/streams/http/child_process shims. |
| `packages/worker/src/runtime/os-contracts.ts` | Shared Runtime OS contracts for filesystem/process/ports/package ABI/diagnostics. |
| `packages/worker/src/runtime/sqlite-runtime-fs-bridge.ts` | Runtime filesystem bridge over `SqliteVFS`. |
| `packages/worker/src/facets/process.ts` | Supervisor-side `child_process` broker. |
| `packages/worker/src/loaders/process-fabric.ts` | Resident-process scheduler, boot specs, and `openResidentFacet` — the one way a process becomes a running facet. |
| `packages/worker/src/loaders/process-host.ts` | Which actor hosts that facet: the user's own session DO, or a sibling. Two implementations, one deployment-wide choice. |
| `packages/worker/src/vfs/sqlite-vfs.ts` | SQLite-backed VFS. |
| `packages/worker/src/npm/installer.ts` | npm install pipeline. |
| `packages/worker/src/runtime/package-manager.ts` | `nimbus install` runtime package manager. |

Constants live in `packages/worker/src/constants.ts`.

Node dynamic Workers keep sync `fs` calls on a startup snapshot and local write
cache for speed. The snapshot includes the entry dependency graph plus a bounded
current-working-tree project snapshot, excluding `node_modules`, `.git`, and
`.nimbus`. Async `fs` calls use the supervisor bridge for live SQLite VFS reads
and common mutations (`writeFile`, `appendFile`, `mkdir`, `unlink`, `rename`,
`rmdir`, `symlink`, `readlink`, `truncate`), while merging live directory
entries so child-process writes are visible inside long-running Node
processes. `fs.promises.open` FileHandles and live appends use the stateless
range RPCs (`fsReadRange`/`fsWriteRange`/`fsTruncate`), which rewrite only the
touched 64 KiB chunks; VFS revisions are per-path subtree watermarks
(`SqliteVFS.revision(path?)`).

Every resident process — node servers, python/ruby socket servers, the opencode
TUI and its headless server — is a DO Facet named `proc-<pid>`. **Which actor
hosts that facet is one deployment-wide var, `NIMBUS_PROCESS_HOST`:**

| Value | Where the process runs | Spawn | Memory | CPU |
|---|---|---|---|---|
| `facet` (default) | a child actor of the user's own session DO | ~250 ms p50 | independent | shared with siblings |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AshishKumar4/Nimbus](https://github.com/AshishKumar4/Nimbus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
