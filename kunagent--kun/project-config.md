---
trigger: always_on
description: This guide is for AI agents working in this repository. It collects the project facts, recurring pitfalls, and validation paths that should be checked before making changes.
---

# DeepSeek-GUI Agent Guide

This guide is for AI agents working in this repository. It collects the project facts, recurring pitfalls, and validation paths that should be checked before making changes.

## Project Boundaries

- This is an `Electron + React + TypeScript` desktop app. The product name is `Kun`; the top-level build entries are `package.json`, `electron.vite.config.ts`, and `electron-builder.config.cjs`.
- The only active agent runtime is the bundled `kun/` TypeScript package. The GUI talks to `kun serve` over local `HTTP + SSE`; the renderer does not run the agent loop directly.
- The main data path is `Renderer -> preload -> main -> Kun runtime`:
  - `src/renderer/src`: React workbench, Code/Design/Write/Connect phone UI, and Zustand state.
  - `src/preload`: the constrained IPC bridge exposed to the renderer as `window.kunGui`.
  - `src/main`: Electron main process, windows, system services, settings, runtime host, and IPC handlers.
  - `src/shared`: cross-layer types, settings normalizers, and provider contracts.
  - `kun/`: `kun serve`, HTTP routes, thread/session stores, tool execution, model adapters, agent loop, cache, and usage tracking.
- The single-runtime plan and the legacy surfaces that must stay removed are documented in `docs/AGENTS.md` and `docs/kun-architecture.md`. Read both before changing runtime, agent selector, diagnostics, or legacy provider behavior.

## Code Organization

- Put new UI interactions in `src/renderer/src`, Electron/system integration in `src/main`, cross-layer contracts in `src/shared`, and agent runtime behavior in `kun/src`.
- For new IPC or runtime APIs, define the shared schema/types first, then wire preload bridge, main handler/runtime adapter, and renderer consumer.
- For runtime HTTP/SSE issues, start with:
  - `src/main/runtime/kun-adapter.ts`
  - `src/main/runtime-sse-ipc.ts`
  - `src/preload/index.ts`
  - `src/renderer/src/agent/kun-runtime.ts`
  - `src/renderer/src/agent/kun-mapper.ts`
  - `kun/src/server/runtime-factory.ts`
  - `kun/src/server/routes/`
- Do not recreate old runtime paths, provider switchers, runtime diagnostics panels, CodeWhale/Reasonix adapters, process managers, or RPC bridges. Legacy provider fields should only be read during settings migration and folded into `agents.kun`.
- Connect phone code may still use the internal `claw` name for compatibility. That name does not mean there should be a separate Claw runtime.

## Providers And Model Requests

- Endpoint behavior is a shared contract, not a local detail of one caller. When changing `baseUrl`, `endpointFormat`, request bodies, or provider presets, inspect all relevant consumers:
  - `src/shared/openai-compat-url.ts`
  - `src/shared/app-settings-provider.ts`
  - `src/shared/app-settings-kun.ts`
  - `src/main/upstream-models.ts`
  - `src/main/provider-connection.ts`
  - `src/main/services/write-inline-completion-service.ts`
  - `src/main/claw-scheduled-task-detector.ts`
  - `kun/src/adapters/model/deepseek-compat-model-client.ts`
  - `kun/src/loop/agent-loop.ts`
- Blank `baseUrl` values fall back to `DEFAULT_DEEPSEEK_BASE_URL = https://api.deepseek.com`. Keep URL path appending separate from JSON body fields when explaining or changing request behavior.
- Supported endpoint families include OpenAI chat completions, Anthropic messages, and OpenAI responses compatibility modes. `endpointFormat` affects URL construction, headers, request body shape, stream parsing, usage parsing, and reasoning fields.
- A custom full endpoint path is an explicit mode. Do not guess that the user-provided URL should receive another appended path.
- For model request 404s, user-facing guidance should point at provider configuration, especially Base URL and Endpoint format. Kun logs should include sanitized `baseUrl`, final `requestUrl`, provider, model, `endpointFormat`, HTTP status, and a summarized response body.
- Write inline completion, plan/scheduled-task detection, model listing, provider probing, and the main chat loop can use different request bodies. Do not inspect only the chat path and then generalize to every model request.

## Kun Runtime Notes

- `kun/src/server/runtime-factory.ts` is the runtime composition root. In production serve mode, the event bus, thread/session stores, and runtime event recorder are wired there.
- `RuntimeEventRecorder`, `FileSessionStore`, and `InMemoryEventBus` are the key paths for event persistence, SSE replay, and memory investigation. For OOM or event growth issues, inspect in-memory arrays, `events.jsonl`, `highestSeq()`, and replay behavior together.
- The agent loop is in `kun/src/loop/agent-loop.ts`. Tool calls, approvals, user input, plan/goal handling, model-history repair, usage, and cache accounting can all pass through it.
- The GUI HTTP surface is broader than chat: thread list/search/archive, fork, resume-thread, approvals, user-inputs, usage, and workspace status all need to remain equivalent when runtime code is changed.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KunAgent/Kun](https://github.com/KunAgent/Kun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
