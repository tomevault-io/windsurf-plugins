---
trigger: always_on
description: **Package:** `@saintno/comfyui-sdk` — TypeScript SDK for ComfyUI server communication.
---

# AGENTS.md

## Project Overview

**Package:** `@saintno/comfyui-sdk` — TypeScript SDK for ComfyUI server communication.
**Runtime:** Bun (test runner, bundler, package manager). No Node.js or npm.
**Output:** Dual bundle — `build/index.esm.js` (browser) + `build/index.cjs` (Node), with `build/index.d.ts`.

## Commands

```bash
bun run build          # Build dist bundles + types
bun test               # Unit tests only (integration auto-skipped)
bun run test:integration # Full suite against http://192.168.14.93:8188
COMFYUI_HOST=http://host:8188 bun test  # Run with custom server
```

## Architecture

```
src/
├── client.ts          # ComfyApi — main entry point, extends EventTarget
├── socket.ts          # WebSocketClient — auto-picks native ws vs Node ws
├── call-wrapper.ts    # CallWrapper — runs workflows, emits event callbacks
├── prompt-builder.ts  # PromptBuilder — fluent API to mutate workflow JSON
├── pool.ts            # ComfyPool — multi-client job distribution
├── tools.ts           # Pure utility functions (seed, randomInt, delay, encodePath)
├── contansts.ts       # Constants (polling interval, timeouts)
├── features/
│   ├── abstract.ts    # AbstractFeature base class
│   ├── manager.ts     # ManagerFeature — ComfyUI-Manager extension (may not be installed)
│   └── monitoring.ts  # MonitoringFeature — Crystools extension (may not be installed)
└── types/
    ├── api.ts         # Core types (QueueResponse, SystemStats, ServerFeatures, etc.)
    ├── event.ts       # Event maps (TComfyAPIEventMap, TComfyPoolEventMap)
    ├── error.ts       # Error types
    ├── manager.ts     # Manager-specific types
    ├── sampler.ts     # Sampler/scheduler name union types
    └── tool.ts        # PromptBuilder config types
```

## Key Patterns

### Error Handling

All client methods use a `this.log(fnName, message, data)` pattern that dispatches a `"log"` event instead of `console.log`. Errors are logged then re-thrown.

### API Communication

`fetchApi(route, options)` is the internal HTTP wrapper. It auto-injects credentials (basic/bearer/custom headers) and handles host concatenation. Never call `fetch` directly.

### Feature System

Extensions live under `api.ext.manager` and `api.ext.monitor`. Each extends `AbstractFeature` and must implement `checkSupported()` and `destroy()`. Features may not be installed on every server — always check `isSupported` before calling methods.

### PromptBuilder

Uses immutable-style chaining. Methods like `bypass()`, `input()`, `setInputNode()` return a modified clone. The original instance is not mutated (except `inputRaw()`).

### WebSocket Fallback

If WebSocket connection fails, `createSocket()` falls back to HTTP polling via `setupPollingFallback()`.

## API Status

### Stable (use these)

`ping`, `pollStatus`, `getSystemStats`, `getExtensions`, `getEmbeddings`, `getNodeDefs`, `getCheckpoints`, `getLoras`, `getSamplerInfo`, `getQueue`, `queuePrompt`, `getHistory`, `getHistories`, `getSettings`, `getSetting`, `storeSetting`, `storeSettings`, `getUserData`, `storeUserData`, `deleteUserData`, `moveUserData`, `interrupt`, `freeMemory`, `getImage`, `getPathImage`, `uploadImage`, `uploadMask`, `getTerminalLogs`, `setTerminalSubscription`, `getUserConfig`, `createUser`, `getFeatures`, `getModelTypes`, `getModels`, `getWorkflowTemplates`, `getViewMetadata`, `clearHistory`, `manageQueue`, `listUserDataV2`

### Deprecated (still works, has replacement)

| Method                  | Replacement                                                             |
| ----------------------- | ----------------------------------------------------------------------- |
| `listUserData()`        | `listUserDataV2()` — returns structured entries with type/size/modified |
| `getModelFolders()`     | `getModelTypes()` — stable `/models` endpoint                           |
| `getModelFiles(folder)` | `getModels(folder)` — stable `/models/{folder}` endpoint                |

### Removed (endpoint no longer exists in ComfyUI 0.18.1)

`getModelPreview()`, `getModelPreviewUrl()` — `/experiment/models/preview/*` was removed upstream.

### Experimental

`setTerminalSubscription` — `/internal/logs/subscribe` may return 500 on some deployments.

## Testing

### Pattern

- **Framework:** `bun:test` (no jest/vitest)
- **Unit tests:** `test/*.spec.ts` — no network, always run
- **Integration tests:** `test/*.integration.spec.ts` — require live ComfyUI server

### Gating

Integration tests use `describe.skipIf(!process.env.COMFYUI_HOST())` at the top level. Without `COMFYUI_HOST`, they are **skipped**, not failed. Never set `describe.skipIf` per-test; only at the `describe` block level.

### Fixtures (`test/fixtures.ts`)

```ts
import { getTestHost, createTestClient, waitForClient } from "./fixtures";
```

- `getTestHost()` — returns `process.env.COMFYUI_HOST || ""`
- `createTestClient(clientId?)` — creates `new ComfyApi(getTestHost(), clientId)`
- `waitForClient(client)` — calls `client.init(5, 2000).waitForReady()`

### Timeouts

- Default: bun:test default (5s)
- Integration suites: no describe-level timeout (set per-test via 3rd arg)
- Workflow execution tests: `it("name", fn, 120_000)`
- Pool tests: `it("name", fn, 30_000)`

### Lifecycle

```ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [comfy-addons/comfyui-sdk](https://github.com/comfy-addons/comfyui-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
