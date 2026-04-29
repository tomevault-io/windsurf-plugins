---
trigger: always_on
description: Entrypoint for any coding agent working in this repo.
---

# AGENTS.md

Entrypoint for any coding agent working in this repo.

`CLAUDE.md` is a symlink to this file. Both point to the same content.

## What This Project Is

Skelegent is a Rust workspace implementing a 6-layer composable agentic AI
runtime. Layer 0 defines the stability contract (protocol traits, wire types).
Layers 1–5 build implementations on top. Every concern — from provider
serialization to secret management — lives in exactly one crate.

Core values (in priority order): composability over convenience, declaration
separated from execution, slim defaults with opt-in complexity. See
`ARCHITECTURE.md` for full rationale.

## Key Abstractions

You must understand these types to work in this codebase:

| Type                                      | Crate              | Role                                                                                                                                                                                                                                          |
| ----------------------------------------- | ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Operator`                                | layer0             | Object-safe trait. `execute(input, ctx) -> Result<OperatorOutput, ProtocolError>`. The unit of agent behavior.                                                                                                                                |
| `DispatchContext`                         | layer0             | Execution metadata threaded through every boundary: dispatch ID, trace context, auth, typed extensions. Every operator, tool, and middleware receives this.                                                                                   |
| `Context`                                 | skg-context-engine | Mutable conversation substrate: messages, extensions, metrics, intents. Direct synchronous mutations. Intents declared via `push_intent()` and drained into `OperatorOutput::intents`.                                                        |
| `Middleware` / `Pipeline`                  | skg-context-engine | `Middleware` is the single abstraction for context transformation. `Pipeline` holds ordered before_send/after_send middleware phases. Budget guards, compaction, telemetry are all middleware.                                                  |
| `Intent`                                  | layer0             | Executable declarations (Delegate, Handoff, Signal, WriteMemory, etc.). Operators declare intent; outer layers execute.                                                                                                                        |
| `ExecutionEvent`                          | layer0             | Semantic observation envelope: status changes, tool calls, intent declarations, artifacts, completion. Stream-first.                                                                                                                           |
| `CapabilitySource` / `CapabilityDescriptor` | layer0           | Read-only discovery. Sibling to `Dispatcher`. Describes what operators are available and what they accept.                                                                                                                                    |
| `Outcome`                                 | layer0             | Typed invocation result: Terminal, Suspended, Transferred, Limited, Intercepted.                                                                                                                                                              |
| `ProtocolError`                           | layer0             | Canonical serializable failure at invocation boundaries.                                                                                                                                                                                      |
| `Provider`                                | skg-turn           | NOT object-safe. Generic `<P: Provider>` everywhere, erased at the `Operator` boundary. Wraps LLM inference (Anthropic, OpenAI, Ollama, etc.).                                                                                               |
| `Dispatcher`                              | layer0             | Invokes operators by ID. The orchestration boundary.                                                                                                                                                                                          |

### How they connect

```
User message
  → Dispatcher.dispatch(ctx: &DispatchContext, input)
    → Operator.execute(input, &DispatchContext)
      → react_loop(Context, Provider, Tools, &DispatchContext, config, &Pipeline)
        → Pipeline.run_before(ctx) runs middleware
        → Context.compile() + Provider.infer(request)
        → Pipeline.run_after(ctx) runs middleware
        → Provider.infer(request) → response (projected into ExecutionEvents)
        → Tools execute with DispatchContext
        → Context.push_intent(Intent) declares executable intent

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [secbear/skelegent](https://github.com/secbear/skelegent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
