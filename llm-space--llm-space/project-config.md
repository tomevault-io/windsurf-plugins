---
trigger: always_on
description: A workbench for prompt and agent development — build, trace, debug, evaluate, and manage, all in one place. It ships as a native **desktop app** (Electrobun), not a website.
---

## Introduction

A workbench for prompt and agent development — build, trace, debug, evaluate, and manage, all in one place. It ships as a native **desktop app** (Electrobun), not a website.

## Tooling

Use **bun** for everything (`packageManager: bun`, pinned to `bun 1.3` in `mise.toml`). Do not use npm/pnpm/yarn.

| Task | Command | Notes |
|---|---|---|
| Install deps | `bun install` | from repo root |
| Run desktop app | `bun dev` | root script → `cd apps/desktop && bun run dev:hmr` (Vite HMR on :5173 + `electrobun dev --watch`) |
| Run desktop app with CEF/CDP debugging | `bun run dev:cef` | root script → `cd apps/desktop && bun run dev:cef`; exposes CDP on `127.0.0.1:9333` by default |
| Build (canary) | `bun run build:canary` | in `apps/desktop` → `vite build && electrobun build --env=canary` |
| Lint | `bun lint` / `bun run lint:check` | `lint` = `eslint --fix`, `lint:check` / `check` = `eslint .`; flat config at repo root |
| Add a dependency | `bun add <pkg>` | run inside the target package (`apps/desktop` or `packages/core`) |
| Add a shadcn/ui component | `bunx --bun shadcn@latest add <component>` | run inside `apps/desktop` |
| Run a script from root | `bun --filter <pkg> <script>` | e.g. `bun --filter @llm-space/desktop start` |

There is **no test framework** and **no root typecheck script**; each package uses `tsc` via `tsconfig.json`.

Shared dependency versions live in the root `package.json` `catalog` (referenced as `"catalog:"`) — bump them there, not per-package. The catalog currently pins `@earendil-works/pi-ai`, `@earendil-works/pi-agent-core`, `react`, `react-dom`, and `typebox`.

### Electrobun page debugging

When you need to inspect or debug the real desktop renderer, use the project
skill at `./.agents/skills/electrobun-cdp-debug/SKILL.md`. Do **not** mock
`electrobun.rpc` in a browser.

Start with `bun run dev:cef`; normal `bun dev` keeps the native WebView renderer
and does not expose CDP.

When CEF/CDP verification needs an isolated app data root, put runtime sandbox
data in the system temporary directory by default, not under `.agents/` or the
repo:

```sh
TMP_ROOT="$(mktemp -d "${TMPDIR:-/tmp}/llm-space-XXXXXX")"
LLM_SPACE_ROOT="$TMP_ROOT" bun run dev:cef
```

Only keep durable evidence in the repo, such as audit screenshots, notes, logs,
and small redacted JSON snippets. Do not commit or leave routine `workspace/`,
`settings/`, caches, or generated app data under `.agents/kaizen-loop/` unless a
fixture is intentionally preserved for review and the reason is documented.

## Architecture

Bun-workspace monorepo. Workspaces are `packages/*` and `apps/*`.

- **`@llm-space/core`** (`packages/core`) — domain library, **no build step**; its TypeScript is consumed directly via the `exports` map. Entrypoints:
  - `.` → re-exports `./client`, `./types`, `./utils`.
  - `./client` — browser-safe pieces: the `streamThread()` client (`client/api`), the `reduceMessages()` streaming reducer (`client/reducer`), and the `AgentTransport` interface (`client/transport`).
  - `./server` — Node/Bun-only implementations: `streamAgent()` (`server/agent/stream`), filesystem paths (`server/paths` — `getLlmSpaceRoot()`, `getSettingsDir()`), `LocalFileSystem` thread storage (`server/storage`), and window-state persistence (`server/window-state`).
  - `./types` — `Thread`/`Message`/`ModelConfig`/`Tool`/`FileNode`/`ModelProviderGroup` and the converters to/from the `@earendil-works/pi-*` formats.
- **`@llm-space/desktop`** (`apps/desktop`) — the Electrobun app. Built with Vite (React 19) for the renderer and `electrobun` for the shell. Two runtime contexts bridged by a single typed RPC channel:
  - **bun main process** (`src/bun/`) — owns the native window, menu, filesystem, model config, and agent streaming.
  - **webview renderer** (`src/app`, `src/components`, `src/mainview`) — the React UI.

### The RPC bridge

The typed contract lives in `src/shared/rpc.ts` (`DesktopRPCType`). The bun side defines handlers in `src/bun/rpc/index.ts` (`mainWindowRPC`); the renderer holds the client in `src/lib/electrobun.ts` (`electrobun.rpc`). Two directions:
- **requests** (webview → bun, request/response): `availableModels`, `addProvider`/`updateProvider`/`removeProvider`/`setModelEnabled`/…, and the filesystem ops `fsLs`/`fsRead`/`fsWrite`/`fsMkdir`/`fsCp`/`fsMv`/`fsRm`/`fsReveal` (mirroring what were HTTP routes).
- **messages** (fire-and-forget, both ways): agent streaming (`sendStreamThreadRequest` / `receiveStreamThreadResponse` / `abortStreamThread`), fullscreen sync, and `executeCommand` (see the command layer).

Electrobun RPC has no native streaming, so agent runs **simulate a stream over fire-and-forget messages**, correlated by a per-run `streamId` (uuid):
1. Renderer `createRpcTransport()` (`src/client/rpc-transport.ts`) sends `sendStreamThreadRequest { streamId, request }`.
2. Bun `runStreamThread` (`bun/streaming/stream-thread.ts`) iterates `streamAgent()` and sends back `receiveStreamThreadResponse` messages keyed by `streamId`: one `{ type: "event" }` per event, then a terminal `{ type: "done" }` or `{ type: "error", message }`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [llm-space/llm-space](https://github.com/llm-space/llm-space) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
