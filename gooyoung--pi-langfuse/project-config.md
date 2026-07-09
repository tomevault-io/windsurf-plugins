---
trigger: always_on
description: This repository contains a Pi Coding Agent extension that sends Pi runtime telemetry to Langfuse.
---

# AGENTS.md

## Purpose

This repository contains a Pi Coding Agent extension that sends Pi runtime telemetry to Langfuse.
The code is small, event-driven, and stateful. Most changes affect lifecycle ordering, payload shaping,
or session isolation rather than UI behavior.

Use this file as the working guide for code changes in this repo. For user-facing installation and
feature documentation, prefer `README.md` and `README_CN.md`.

## Repository Map

- `index.ts`: extension entrypoint; registers Pi commands and hooks all Pi lifecycle events.
- `src/handlers/agent.ts`: starts and finishes the root Langfuse agent observation and trace IO.
- `src/handlers/generation.ts`: tracks provider requests, response metadata, TTFT, and generation completion.
- `src/handlers/tool.ts`: tracks tool observations, correlates them by `toolCallId`, and records tool error scores.
- `src/handlers/turn.ts`: creates turn-level span wrappers so generations and tools can nest under a turn.
- `src/state.ts`: session-scoped mutable runtime state built on `AsyncLocalStorage`.
- `src/config.ts`: config loading, first-run setup UI, and config persistence.
- `src/langfuse.ts`: Langfuse runtime bootstrap, score client, flush/shutdown, and REST fallback ingestion.
- `src/utils.ts`: payload shaping, truncation, extraction helpers, and defensive parsing.
- `src/constants.ts`: payload size and truncation limits.
- `src/types.ts`: shared runtime and observation typings.
- `test/state.test.ts`: verifies per-session state isolation and overlapping async session safety.
- `test/utils.test.ts`: verifies payload shaping limits and circular handling.
- `.agents/skills/langfuse/`: local Langfuse skill docs and references used by agents working in this repo.

## Runtime Model

The extension maps Pi events onto one Langfuse trace tree:

- One Pi agent run becomes one `pi-agent` trace with a root `agent` observation.
- Provider requests become `llm-generation` observations.
- Tool calls become `tool` observations.
- Turns become `span` observations that can parent generations and tool calls.
- Session-level bookkeeping is keyed by Pi session ID, not by global process state.

The main event flow is:

1. `session_start`: ensure config and reset run state for the session.
2. `before_agent_start` / `agent_start`: create the root agent observation if missing.
3. `turn_start`: open a turn span.
4. `before_provider_request`: start a generation.
5. `after_provider_response`: attach provider metadata and early error status.
6. `message_update`: record TTFT and capture the latest assistant output.
7. `message_end`: finalize the active generation.
8. `tool_execution_start` / `tool_call`: start a tool observation.
9. `tool_result` / `tool_execution_end`: finalize the matching tool observation.
10. `turn_end`: close the turn and synthesize a fallback generation if Pi skipped normal generation events.
11. `agent_end`: close the root observation, update trace IO, and send aggregate scores.
12. `session_shutdown`: close dangling observations and flush Langfuse runtime state.

## Working Rules

- Preserve session isolation. `src/state.ts` uses `AsyncLocalStorage` so overlapping handlers do not leak
  counters or active observations across Pi sessions.
- Preserve idempotency around lifecycle hooks. `before_agent_start` and `agent_start`, and similarly
  tool/generation start-end pairs, may both fire; handlers are written to tolerate duplicate entry points.
- Keep tool correlation keyed by `toolCallId`. This is important for concurrent tool execution.
- Maintain defensive payload shaping. Large objects, circular references, deep trees, and JSON-like strings
  are intentionally normalized before being sent to Langfuse.
- Do not bypass `shapePayload()`, `truncate()`, or related helpers when adding new telemetry fields.
- Treat config and credentials as sensitive. Never hardcode keys or commit local config artifacts.
- Prefer minimal metadata additions. Langfuse payloads should stay readable and bounded.

## Config Behavior

Config precedence is:

1. `~/.pi/agent/pi-langfuse/config.json`
2. `LANGFUSE_PUBLIC_KEY` / `LANGFUSE_SECRET_KEY`
3. `LANGFUSE_BASE_URL` or `LANGFUSE_HOST`
4. Interactive `/langfuse-setup` in Pi UI when config is missing

Relevant implementation details:

- `src/config.ts` loads saved config first, then env vars.
- First-run setup is only attempted once per session via `state.setupAttemptedThisSession`.
- Manual `/langfuse-setup` clears cached config and shuts down the runtime before reconfiguring.

## Langfuse-Specific Notes

- The runtime is created lazily in `src/langfuse.ts`.
- OpenTelemetry export is the primary path.
- If OTel accepts spans but the trace never becomes visible, the extension falls back to Langfuse REST ingestion.
- Scores are sent separately through the Langfuse client; they are not part of the OTel span export path.
- Root trace IO is mirrored from the root agent observation when `setTraceIO()` is available.

When editing Langfuse integration code, be careful with:

- flush and shutdown ordering
- trace visibility polling and fallback ingestion
- observation parent/child nesting
- score attribution to trace vs observation IDs

## Change Guidance

### When editing lifecycle code


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gooyoung/pi-langfuse](https://github.com/gooyoung/pi-langfuse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
