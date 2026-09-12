---
trigger: always_on
description: Guidance for working in this repo. Start with [README.md](README.md) (layout +
---

# AGENTS.md

Guidance for working in this repo. Start with [README.md](README.md) (layout +
how to run) and [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) (seams, event model,
design decisions) — this file is the operational overlay, not a duplicate.

## The one-line mental model

A stateful agentic inbox running one runtime (DeepAgents/LangGraph). The runtime
emits native
`@langchain/protocol` `ProtocolEvent` frames via `streamEvents(v3)`; every
frontend consumes them through the `@langchain/langgraph-sdk`
`Client`/`ThreadStream`/`StreamController` over HTTP/SSE. The seam is that
**protocol + SDK projection** boundary; keep React coupling outside it.

## Layering discipline (don't break these)

- `packages/core` is **nearly pure** — no `node:*`, no DOM, no `deepagents` /
  `@langchain/langgraph`, no `ai` — but it MAY import `@langchain/core` model/agent
  **types** (`BaseChatModel`), because the app is coupled to LangGraph by design
  and the model seam is honestly typed, not laundered through `unknown`. Core also
  owns the pure protocol/wire types (`protocol-types.ts`). The framework-agnostic
  message/thread-slice projection helpers live in `apps/web/src/projection` — the
  only consumer is the web app, so they sit beside it rather than in a shared
  package. The eslint layering rule enforces core's purity.
- `packages/plugin-api` is the public, browser-safe plugin contract — no
  `node:*`, LangChain, runtime, or UI imports. `packages/plugin-sdk` owns the
  Node-bound filesystem loader, materializers, MCP clients, and contribution
  registries. The eslint layering rule enforces plugin-api's purity.
- **`packages/runtime-langgraph` is the ONLY production package that imports
  `deepagents` — the graph engine.** The `tests/langgraph-compat` workspace is a
  test-only exception because it pins DeepAgents' public exports. If the
  DeepAgents API churns, the runtime blast radius must stay one package. In
  production, two narrow, deliberate exceptions touch
  `@langchain/langgraph` WITHOUT the engine: `packages/storage` imports its
  `InMemoryStore` (a checkpoint-store primitive, kept out of runtime-langgraph
  because it's Node-bound), and `apps/api-server` imports `ProtocolEvent` as a
  TYPE only (the wire event). The conformance workspace may import upstream
  types for tests. The eslint layering guards enforce the core/ui purity +
  frontend rules; the `deepagents` "only" invariant is a convention, not
  lint-enforced.
- The frontend (`apps/web`) never imports a runtime or a model
  binding. It MAY import the transport SDK (`@langchain/langgraph-sdk`) — that's
  the wire — but never the runtime graph engine.
- The api-server assembles the concrete runtime through
  `createPizzaBotAgent` (`packages/runtime-langgraph`). Core knows only the
  minimal structural `AgentHandle` (`core/src/agent-run.ts`) the concrete agent
  satisfies (state read/write; the streaming method `streamProtocol` lives on
  the concrete `LangGraphAgent`, since its `ProtocolEvent`s can't be named
  without a `@langchain/langgraph` import). The runtime is stateful and
  checkpointer-backed.

## Where things live (for the common asks)

- **Runtime → protocol stream** (LangGraph → `ProtocolEvent` frames):
  `runtime-langgraph/src/stream-protocol.ts` holds `streamProtocolEvents` +
  `toLangGraphInput`; `index.ts`'s `LangGraphAgent.streamProtocol()` is the thin
  driver over it and the only streaming path. Native v3 correlation is
  reassembled client-side by the SDK `StreamController`. The offline conformance
  guard is `tests/langgraph-compat/protocol-stream-conformance.test.ts`.
- **The protocol/wire types**: `packages/core/src/protocol-types.ts`
  (`NormalizedMessage`, `ThreadStateValues`, HITL vocab, `ErrorCode`, `RunStatus`,
  `PROTOCOL_VERSION`, and wire schemas).
- **Runtime factory + agent handle**: `runtime-langgraph`'s `index.ts` exports
  `createPizzaBotAgent(systemPrompt, deps)` returning a `LangGraphAgent`; the run data
  types + the structural `AgentHandle` it satisfies live in
  `packages/core/src/agent-run.ts`.
- **Server composition root** (wires the runtime factory + storage + model):
  `apps/api-server` (`agent-host.ts` is the assembly; `index.ts` serves it). The
  in-process run registry is `ProtocolRunManager` (`protocol-run-manager.ts` —
  buffers frames per run, fans out to observers with `since`-based replay,
  cancels via `AbortSignal`); the SDK-facing wire is `routes-protocol.ts` (4
  Agent-Protocol endpoints: `POST /threads/:id/commands`, `POST
  /threads/:id/runs/:run_id/cancel` — what the SDK's `stop()` calls — `POST
  /threads/:id/stream/events`, `GET /threads/:id/state`).
- **Message/thread projection** (SDK projections → feed messages/cards): the
  pure helpers in `apps/web/src/projection` (`messages.ts` =
  `messagesToUI`/interrupt overlays/hydrate helpers, `thread-slice.ts` =
  `ThreadSlice`/`StreamStatus`). The web binds via
  `apps/web/src/protocol-stream-store.ts` (a headless SDK `StreamController` per
  thread) behind the thin `use-thread-slice.ts` hook;
  `components/ActivityRail.tsx` renders delegated work in the Activity panel.
- **Model providers**: `packages/inference-providers` (one adapter per provider

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pizza-bot-app/pizza-bot](https://github.com/pizza-bot-app/pizza-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->
