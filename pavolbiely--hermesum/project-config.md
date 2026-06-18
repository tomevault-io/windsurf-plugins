---
trigger: always_on
description: Hermesum is a Nuxt/Nitro web chat interface for Hermes Agent. The integration is ACP-native: the browser talks to same-origin Nitro routes, Nitro owns a long-lived `hermes acp` subprocess through the official Agent Client Protocol TypeScript SDK, and Hermes Agent remains the actual agent runtime.
---

# AGENTS.md

## Project Overview

Hermesum is a Nuxt/Nitro web chat interface for Hermes Agent. The integration is ACP-native: the browser talks to same-origin Nitro routes, Nitro owns a long-lived `hermes acp` subprocess through the official Agent Client Protocol TypeScript SDK, and Hermes Agent remains the actual agent runtime.

Treat this repository as the source of truth for Hermesum work. Do not edit `$HOME/.hermes/hermes-agent` directly unless the user explicitly approves it.

## Repository Map

- `web/`: Nuxt 4 app using Nuxt UI, Comark, Nitro server routes, and `@agentclientprotocol/sdk`.
- `web/app/pages/index.vue`: new-chat entry point and initial ACP session creation.
- `web/app/pages/acp/[id].vue`: main ACP-native chat route.
- `web/app/layouts/default.vue`: app shell, sidebar, workspace/profile controls, settings, and navigation.
- `web/app/components/`: chat, prompt, sidebar, workspace, read-aloud, and utility UI components.
- `web/app/composables/`: typed ACP/app API clients and local browser UI state.
- `web/app/types/`: frontend/API contract types for ACP and UI chat state.
- `web/app/utils/`: ACP normalization re-exports, sidebar mapping, queued messages, drafts, sounds, highlighting, and UI helpers.
- `web/shared/acp/`: browser/server-safe ACP event and transcript normalization helpers.
- `web/server/acp/`: ACP bridge, event backlog, replay capture, permission handling, prompt metadata, and runtime helpers.
- `web/server/api/acp/`: ACP protocol-backed Nitro routes for health, initialize, sessions, prompts, cancellation, metadata, permissions, config, and event streaming.
- `web/server/app/` and `web/server/api/app/`: Hermesum-owned product features that ACP does not own, including workspaces, profile listing, session metadata, and read-aloud speech.
- `web/tests/`: `node:test` coverage for shared helpers, transcript/event normalization, sidebar mapping, workspaces, queued messages, read-aloud, and related utilities.
- `web/nuxt.config.ts`: runtime config for ACP command/args/cwd; defaults to `hermes --profile hermesum acp`.
- `run-local.sh`: local Nuxt dev/preview orchestration.
- `.hermes/agent-map.md`: first-read navigation map for future agents.
- `.runtime/`: disposable generated runtime/cache state. Do not treat it as source code.

## Core Engineering Rules

- Prefer small, clear, maintainable changes over clever or broad rewrites.
- Understand existing flow before changing code, especially ACP bridge state, SSE streaming, session replay, permission handling, and prompt correlation.
- Keep code reusable where reuse is real, but avoid generic abstractions for one-off prototype code.
- Favor focused modules, typed boundaries, explicit validation, and predictable error handling.
- Prefer project-native and framework-native APIs before adding dependencies.
- Do not add new large catch-all files. When a file starts mixing unrelated concerns or grows past a comfortable review size, split it into cohesive modules before adding more behavior.
- Keep the source tree clean: do not commit generated `.nuxt`, `.output`, `node_modules`, runtime copies, logs, or disposable verification artifacts.
- Keep `README.md`, `AGENTS.md`, `.hermes/agent-map.md`, and active `.hermes/plans/*.md` current when behavior, architecture boundaries, setup/workflow, or verification guidance changes. Do not update docs mechanically for tiny local-only edits.

## Architecture Boundaries

- `web/server/acp/` owns protocol/runtime behavior: ACP process lifecycle, session list/create/load/fork/close, prompt/cancel, permission requests, model/mode/config metadata, SSE backlog, replay capture, and prompt metadata supplements.
- `web/server/api/acp/` exposes ACP-backed HTTP/SSE routes. Do not recreate old `/api/web-chat/*` compatibility contracts.
- `web/server/api/app/` and `web/server/app/` own Hermesum product features that ACP does not own: workspaces, profile list, app-owned ACP session metadata, read-aloud speech, and future app-specific features.
- `web/app` consumes only same-origin `/api/acp/*` or `/api/app/*` contracts through typed composables.
- `web/shared/acp/` is the place for normalization code that must be safe on both server and browser.
- Shared request/response shapes must remain aligned between Nitro handlers, frontend composables, and TypeScript UI types.
- Do not wire browser code directly to ACP stdio. Browser ACP interaction must go through Nitro/server routes.

## ACP Runtime Model

- Use the official `@agentclientprotocol/sdk`; inspect installed SDK types/examples before guessing APIs.
- Keep one long-lived ACP subprocess per server process unless a change explicitly requires otherwise.
- Default ACP command is `hermes --profile hermesum acp`. Override with `HERMESUM_PROFILE`, `HERMESUM_ACP_ARGS`, `HERMESUM_ACP_COMMAND`, or `HERMESUM_ACP_CWD` only when intentionally testing another runtime.
- Model correctness with explicit ACP/session identities: `sessionId`, prompt `turnId`, ACP message ids, tool ids, permission request ids, and server event sequences.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pavolbiely/hermesum](https://github.com/pavolbiely/hermesum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
