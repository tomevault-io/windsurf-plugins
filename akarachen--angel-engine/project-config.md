---
trigger: always_on
description: These notes apply to the whole repository, except vendored code that carries its
---

# Angel Engine Agent Notes

These notes apply to the whole repository, except vendored code that carries its
own instructions.

## Core Rule

Keep protocol and platform adaptation in the runtime adapters. Codex-specific,
ACP-specific, Kimi-specific, OpenCode-specific, or desktop-host-specific wire
semantics must be normalized before they reach shared engine state, client
snapshots, NAPI bindings, or desktop UI projection.

Do not fix provider quirks in `state/`, reducer code, `angel-engine-client`,
NAPI, or desktop unless the quirk has already been converted into a
protocol-neutral engine concept by an adapter.

## Packages

- `crates/angel-engine/`
  - Rust engine library and protocol-neutral state machine.
  - Owns protocol-neutral commands, events, state, reducers, capabilities,
    display-message construction, protocol effects, and transport primitives.
  - Shared state/reducer folders must not inspect provider names or raw provider
    JSON shapes. They should operate on `EngineCommand`, `EngineEvent`,
    `ConversationCapabilities`, and protocol-neutral state types only.

- `crates/angel-provider/`
  - Rust provider adapter crate over `angel-engine`.
  - Owns the `ProtocolAdapter` interface plus built-in Codex and ACP
    implementations.
  - Adapter folders:
    - `src/codex/` owns all Codex app-server wire format, hydrate replay
      normalization, request/response/notification decoding, model catalog
      parsing, and Codex-specific command encoding.
    - `src/acp/` owns all ACP wire format, session update decoding, config
      option parsing, tool call mapping, and ACP-specific command encoding.

- `crates/angel-engine-client/`
  - Rust client API over `angel-engine`.
  - Owns client ergonomics, runtime process IO, JSON-RPC line ingestion,
    snapshots, thread/session helpers, and desktop-friendly primitives such as
    `thread_settings`, `reasoning_level`, `model_list`, `available_modes`,
    `set_model`, `set_mode`, and `set_reasoning_level`.
  - It may orchestrate when to initialize, hydrate, inspect, drain, or send
    thread events. It must not parse provider wire formats or invent
    provider-specific behavior. If a runtime needs special interpretation, add
    it to the corresponding `angel-provider` adapter and expose the normalized
    result through engine state.

- `crates/angel-engine-client-napi/`
  - Node.js N-API binding for `angel-engine-client`.
  - This package should stay thin: convert Rust client APIs into JS-callable
    classes/functions and generated TypeScript definitions.
  - Do not add settings policy, protocol normalization, stream merging rules, or
    desktop UI assumptions here. Rebuild it after Rust API/type changes with:
    `npm --prefix crates/angel-engine-client-napi run build`.

- `desktop/`
  - Electron desktop application.
  - Uses `@angel-engine/client-napi` as its backend API.
  - `src/main/` owns Electron main-process services: IPC routing, chat metadata
    persistence, session lifetime, abort handling, and elicitation handoff.
  - `src/main/chat/projection.ts` may map normalized NAPI snapshots/events into
    desktop shared chat types. It must not infer Codex/ACP semantics from raw
    runtime payloads.
  - `src/shared/` owns TypeScript types shared by main and renderer.
  - `src/lib/engine-model-adapter.ts` owns assistant-ui runtime adaptation and
    renderer-side message accumulation.
  - `src/pages/`, `src/chat/`, `src/components/`, `src/app/` own UI only.
    Renderer state may track draft user selections, but available models,
    reasoning levels, modes, and current runtime settings should come from the
    engine/client snapshots.
  - The desktop SQLite database stores chat/project metadata only. It should not
    store messages; restored messages should come from runtime hydrate via the
    engine/client path.

- `crates/test-cli/`
  - Test/support CLI crate. Keep it as a consumer of public APIs rather than a
    place for engine behavior.

- `vendor/agent-client-protocol/`
  - Vendored ACP reference material. Treat as third-party code and avoid edits
    unless intentionally updating the vendored dependency.

## Layer Boundaries

1. Wire format enters and leaves through adapters.
   - Decode provider messages into `EngineEvent`.
   - Encode `ProtocolEffect` into provider JSON-RPC.
   - Normalize provider hydrate/replay quirks before creating history replay
     chunks.
   - For protocol fields with a closed value set, use typed enums from the
     owning crate when available, or serde enums with exact canonical wire names
     in the adapter. Do not add string alias compatibility, casing fallbacks, or
     `contains`/prefix/suffix guessing for protocol ids. Unknown values should
     fail fast so casing drift is caught during development.
   - Missing or malformed boundary input should throw immediately. Do not fill
     missing IDs, text, arrays, objects, or settings with `""`, `[]`, `{}`, or
     other silent defaults unless the provider adapter is explicitly normalizing
     a documented provider quirk into a protocol-neutral concept.

2. Reducers and state are protocol-neutral.
   - Reducers apply `EngineEvent` to state.
   - State/display code may understand common concepts such as turns, actions,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AkaraChen/angel-engine](https://github.com/AkaraChen/angel-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
