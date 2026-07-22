---
trigger: always_on
description: [Root AGENTS.md](../AGENTS.md) | CLI entrypoint, session bootstrap, agent runloop wiring.
---

# vtcode (binary)

[Root AGENTS.md](../AGENTS.md) | CLI entrypoint, session bootstrap, agent runloop wiring.

## Modules

`main.rs` binary entry | `agent/` runloop + subagent dispatch | `cli/` CLI handlers | `startup/` first-run + onboarding | `updater/` self-update | `codex_app_server/` app server bridge | `main_helpers/` tracing + runtime init

## Rules

- Thin binary — all runtime logic in `vtcode-core`. This crate wires CLI args to `Agent::run()`.
- Global allocator is `mimalloc` by default; `allocator-jemalloc` opts into
  `tikv-jemalloc` (background-thread purging, recommended for Linux servers).
  Selected in `src/allocator.rs` (a `mod allocator;` in `src/main.rs`). See
  `docs/development/ALLOCATOR_MEMORY.md`.
- `vtcode bench-allocator` measures RSS under a bursty/sparse Tokio workload to detect
  allocator memory pinning; use it before changing the default allocator (see Gotchas).
- `vtcode_ui::tui::panic_hook` installs custom panic handler — must run before any output.
- `agent/runloop/` contains the single-agent runloop. Multi-agent is in `vtcode-core::subagents`.
- `agent/runloop/unified/turn/compaction/` is a **thin delegator** — all compaction logic (auto/manual orchestration, memory envelope, dedup, thresholds) lives in `vtcode-core::compaction`. Do not re-implement compaction here; call the shared orchestrator.

## Gotchas

- `main_helpers` handles runtime relaunch context — do not duplicate init logic in `main.rs`.
- Allocator memory pinning: vtcode's bursty/sparse Tokio workload (semaphore-capped
  `JoinSet` fans-out, then workers idle) makes both `mimalloc` and `glibc` hold RSS flat
  after a burst (frees stranded on cross-thread lists, only reconciled by future allocation
  activity). `jemalloc` only returns memory on idle when its `background_thread` is active —
  on macOS the jemalloc build prints `background_thread currently supports pthread only` and
  behaves like mimalloc, so switching the allocator does NOT help on macOS dev. On Linux
  containers (the article's target) `background_thread` works and jemalloc reclaims memory.
  Measure with `vtcode bench-allocator` before changing the default; do not switch blindly.
- `load_dotenv()` must run before config load to pick up `.env` API keys.
- Provider noise (e.g. MiniMax `]<]minimax[>[`) is stripped centrally in `turn::provider_noise`. Stream-level sanitization (harmony + minimax) lives in `stream_sanitization::StreamSanitizer`. Do not re-implement noise stripping inline.
- Budget exhaustion (wall-clock via `record_wall_clock_exhaustion_notice`, tool-call via `record_tool_budget_exhaustion_notice`) both follow the same contract: emit the full policy message once, a compact stub for later calls in the batch, then a single "synthesize now" directive via `flush_budget_synthesis_directives` (called after the tool batch so it is never interleaved between tool responses). Do not push the directive inline in `validate_tool_call`, and NEVER make tool-call budget exhaustion `Break` the turn as `Blocked` — that skips the synthesis pass entirely, so plan mode ended with research but no plan and the model looped "I'll synthesize the plan" across continue-turns. The flush also arms `switch_to_tool_free_recovery()` so the next request strips tool definitions at the API level — the directive alone is advisory and models kept emitting rejected tool calls after it (turn_637/turn_647). `tool_budget_exhausted_emitted` joins the wall-clock flags in the plan-mode `mark_recovery_exhausted` gates (`dispatch_post_tool_failure`, turn_loop recovery break).
- `turn_processing/llm_request/` is split into contract-carrying submodules (`snapshot`, `tool_shaping`, `context_management`, `response_chain`, `prompt_assembly`) with `pub(super)` visibility; go through `mod.rs`'s exports. Prompt section order in `prompt_assembly::build_prompt_output` is prompt-cache-sensitive — reordering sections invalidates provider caches. `tool_shaping.rs` carries the wire invariant: hosted Anthropic/OpenAI payloads keep full deferred tool definitions; only the ClientLocal policy omits them (pinned by tests in `request_builder.rs`). `metrics.rs` emits a per-request `token_budget_breakdown` turn metric (system-prompt / tool-schema / message-history tokens + on-wire tool count) to the `vtcode.turn.metrics` target and trajectory log; cache read/write/miss are not duplicated here (they live in `SessionStats` prompt-cache diagnostics).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vinhnx/vtcode](https://github.com/vinhnx/vtcode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
