---
trigger: always_on
description: - Nanocodex is a headless, library-first Rust agents SDK. The public product is
---

# Development instructions

## Product direction

- Nanocodex is a headless, library-first Rust agents SDK. The public product is
  the embeddable API; the CLI and Harbor adapter are examples and evaluation
  boundaries.
- Keep the scope narrow: one supported OpenAI model family, the Responses
  WebSocket API, one owned agent lifecycle, and caller-defined tools. Do not
  introduce provider/model portability or a generic app-server protocol.
- A normal consumer builds an agent, receives `(Nanocodex, AgentEvents)`, sends
  prompts through the cheap handle, and awaits typed `TurnResult`s. Events are
  optional and independent from results.
- Follow-on prompts reuse the session's retained history automatically. Never
  require callers to pass prior messages, response IDs, or tool results back
  into the agent.
- Builders expose deliberate policy. Queue capacities, socket tasks, mutable
  run state, replay bookkeeping, and similar mechanics stay private.

## Workflow

- Follow the active work in `PLAN.md` in order. Build vertical library slices
  with a real consumer; do not accumulate speculative abstractions.
- Prefer deletion and direct ownership over adapters that merely move data.
  Cleanup should materially reduce production or planning surface.
- Use existing project tooling and patterns. Add a dependency only for a
  concrete need in the current slice.
- Add focused deterministic tests for public contracts and demonstrated
  regressions, not for coverage. Compile public examples as part of validation.
- Use `just run` for a live native smoke. Use focused Harbor trials while
  iterating and the full configured `just eval` only for milestone/release
  gates. Never modify benchmark tasks or verifiers to make Nanocodex pass.
- Inspect the exact JSONL, Harbor result, trajectory, and verifier output for an
  eval claim. Separate cold image/bootstrap time from warm agent work.
- Preserve unrelated work. Never commit `.env`, caches, retained jobs, build
  output, or another user's untracked files.

## Codex reference

- Use the local checkout at `~/github/openai/codex/codex-rs` before making an
  architecture or behavior claim about Codex. Do not browse the web or invoke
  OpenAI documentation tooling unless the user explicitly asks.
- Codex is evidence, not an API requirement. Copy relevant invariants and
  operational behavior while keeping Nanocodex's smaller public surface.
- The reviewed upstream checkpoint is
  `openai/codex@35eaf3ffb0bf2001486c68c47a3d946b34d16634`. A parity review must
  inspect every later commit, classify it as port/evaluate/defer/out-of-scope,
  and cite adopted behavior before advancing the checkpoint.

## Workspace boundaries

- `nanocodex-core` owns dependency-light public data: prompts, events, model
  configuration, and complete typed Responses wire/domain types.
- `nanocodex-service` owns behavior at the API boundary: the persistent
  WebSocket, stream processing, retry policy, telemetry, and generic Tower
  service/client.
- `nanocodex-tools` owns code mode, built-in tools, the heterogeneous registry,
  and the public `Tool` trait.
- `nanocodex-mcp` owns MCP transports, background handshake/discovery,
  authenticated connection inputs, deferred tool search, and remote dispatch.
- `nanocodex` composes those crates into the owned agent lifecycle and exports
  the ergonomic builders and common types.
- `nanocodex-macros` implements `#[tool]`. Keep the executable under
  `bin/nanocodex`; do not move CLI behavior into the library.
- Each lower crate must remain useful without importing the higher orchestration
  crate. Avoid circular concepts and leaky socket/runtime types.

## Runtime invariants

- The private spawned driver is the sole owner of mutable conversation, model,
  tool-runtime, and Tower service state. It runs until all command handles are
  dropped.
- One agent reuses its WebSocket, typed history, code-mode runtime, shell
  sessions, stable cache key, and response chain across sequential turns.
- `prompt().await` waits only for command acceptance and returns an independently
  awaitable `Turn`. Prompt queueing order is owned by the driver.
- Client-owned typed history is authoritative. Healthy turns send only the new
  delta with `previous_response_id`; a replacement socket drops that ID and
  replays complete committed history.
- Commit only completed responses. A failed partial response must not execute a
  tool or enter history.
- Preserve `store: false`, stable prompt/cache identity, and byte-stable shared
  prefixes across turns, retries, compaction, and reconnects.
- Cancellation and process cleanup are explicit. Timeout or cancellation must
  terminate subprocess groups and descendants.

## Tower boundary

- One Tower call is one complete streamed Responses attempt, through
  `response.completed` or a typed failure. Do not return success after merely
  sending the WebSocket frame.
- `ResponsesClient<S>` remains generic over the caller's concrete
  `Service<ResponsesAttempt>`; do not box or globalize the service stack.
- The SDK owns one typed retry/reconnect policy. Caller middleware may wrap it
  with deadlines, concurrency, load shedding, tracing, metrics, circuit
  breaking, or error mapping without becoming a second retry owner.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gakonst/nanocodex](https://github.com/gakonst/nanocodex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-19 -->
