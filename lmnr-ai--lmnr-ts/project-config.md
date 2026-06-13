---
trigger: always_on
description: Guidance for Claude Code (claude.ai/code) when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) when working in this repository.

## Project Overview

Laminar TypeScript SDK — a pnpm monorepo publishing `@lmnr-ai/lmnr`, `@lmnr-ai/client`, `@lmnr-ai/types`, and `lmnr-cli`.

## Repository Structure

- `packages/lmnr` — main `@lmnr-ai/lmnr` package: tracing, OpenTelemetry instrumentations, custom framework exporters (Mastra, Vercel AI SDK, OpenAI, etc.).
- `packages/client` — typed HTTP client for the Laminar API (`@lmnr-ai/client`).
- `packages/types` — shared type defs.
- `packages/lmnr-cli` — `lmnr` CLI.

## Development Commands

```bash
pnpm install                    # Install across workspace
pnpm -r build                   # Build all packages
pnpm -r test                    # Run tests in all packages
pnpm -r lint                    # Lint all packages
pnpm check-versions             # Verify package versions are aligned
```

Work inside a specific package with `pnpm --filter @lmnr-ai/lmnr ...` or `cd packages/lmnr && pnpm ...`.

## Mastra Exporter (`packages/lmnr/src/opentelemetry-lib/instrumentation/mastra/`)

Laminar ships its own `MastraExporter` (implements `ObservabilityExporter` from `@mastra/core/ai-tracing`) instead of relying on the `@mastra/laminar` package. Key non-obvious facts:

- **Span-type mapping**:
  - `MODEL_STEP` → Laminar LLM span (one LLM turn; produces `ai.prompt.messages` / `ai.response.text` / `ai.response.toolCalls`).
  - `TOOL_CALL` / `MCP_TOOL_CALL` → Laminar TOOL span.
  - `MODEL_GENERATION` → Laminar DEFAULT span (the whole agent loop, parent of `MODEL_STEP` and `TOOL_CALL` children).
  - `MODEL_CHUNK` → not exported as spans, but reasoning chunks are intercepted to capture thinking text (see bullet below).
- **Mastra `MODEL_STEP.input` is lossy across steps** — after step 0, tool-result messages show up as empty user messages. We reconstruct the real per-step message history by tracking the parent `MODEL_GENERATION` baseMessages + accumulating per-step turns (assistant tool-call + tool tool-result) from child `TOOL_CALL` spans.
- **`TOOL_CALL` children end BEFORE their parent `MODEL_STEP` ends.** Pair them at *step-end time* against the declared `output.toolCalls` by arrival order — do NOT try to pair them when the tool span ends (declared toolCalls are not available yet).
- Tool-result messages are emitted as `{role:"tool", tool_call_id, content:[{type:"tool-result", toolCallId, toolName, output}]}` — this matches what Laminar's backend (`app-server/src/language_model/chat_message.rs`) expects via `InstrumentationChatMessageAISDKToolCall`/`ToolResult`.
- Exporter internals use narrow local interfaces instead of importing `@mastra/core` types, so there is no peer dep on Mastra.
- **`MastraExporter` requires `Laminar.initialize()` to have been called first.** It owns no transport/batching config and creates spans via `getTracerProvider().getTracer(...)` (NOT `trace.getTracer(...)`, which would return a Noop because Laminar deliberately doesn't register globally — see `tracing/index.ts`). Spans flow through Laminar's `LaminarSpanProcessor` automatically on `Span.end()`. If Laminar isn't initialized the tracer is a noop, `startOtelSpan` sees a `NonRecordingSpan` and drops the span after a one-time warn (`warnNotInitializedOnce`). The only options on `MastraExporterOptions` are `realtime` (force-flush after each span end) and `linkToActiveContext`.
- **Mastra-id alignment is done by mutating the live span.** `startOtelSpan` calls `tracer.startSpan(...)`, then `Object.assign`s Mastra-derived `traceId`/`spanId` over the SDK-allocated ones in `otelSpan.spanContext()` and patches both `parentSpanContext` (SDK v2) and `parentSpanId` (SDK v1 alias). Because the span goes through `LaminarSpanProcessor.onStart` *before* the mutation, it stamps `lmnr.span.path` / `lmnr.span.ids_path` using the auto-generated id; `startOtelSpan` overwrites those attributes with the Mastra-derived versions afterwards. The live span is parked in `liveOtelSpanByMastraId` so `applyEndAttributes` / `setStatus` / `recordException` / `end()` all act on the same instance — no `ReadableSpan` is ever manually constructed here, so the `makeSpanOtelV2Compatible` requirement (next bullet) does not apply.
- Any code that manually constructs a `ReadableSpan` (e.g. framework-exporter bridges that synthesize spans from non-OTel sources) MUST call `makeSpanOtelV2Compatible` from `../../tracing/compat` on the result — without it, setting only v2 fields (`parentSpanContext`, `instrumentationScope`) breaks users on OTel SDK v1 (parent-child links and scope info are dropped, traces render flat).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lmnr-ai/lmnr-ts](https://github.com/lmnr-ai/lmnr-ts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->
