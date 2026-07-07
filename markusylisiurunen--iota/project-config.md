---
trigger: always_on
description: Multi-provider LLM streaming library supporting OpenAI, Anthropic, and Google. Provides a unified interface for streaming completions with tool calling, reasoning summaries (extended thinking), and token usage with cost tracking.
---

# Iota

Multi-provider LLM streaming library supporting OpenAI, Anthropic, and Google. Provides a unified interface for streaming completions with tool calling, reasoning summaries (extended thinking), and token usage with cost tracking.

## Architecture

The library is organized around a streaming pipeline that normalizes cross-provider differences behind a unified event interface.

`stream()` in `src/stream.ts` is the main entry point. It first normalizes the conversation context via `normalizeContextForTarget()`, making message history portable across providers by stripping provider-specific metadata and merging system messages. It then validates any tool definitions against a strict JSON Schema subset that all providers support. Finally, it dispatches to a provider-specific implementation based on the model: `streamOpenAI()`, `streamAnthropic()`, or `streamGoogle()`.

Each provider adapter translates the normalized context into the provider's native API format, initiates a streaming request, and processes incoming chunks. Providers don't push events directly to the stream. Instead, they use a `StreamController` which maintains the in-progress `AssistantMessage` state and emits structured events (`part_start`, `part_delta`, `part_end`, etc.) to an `AssistantStream`.

Consumers iterate over the `AssistantStream` using `for await...of` to receive events as they arrive. When the stream completes, calling `result()` returns the final `AssistantMessage` containing the full response content, stop reason, and token usage with cost.

## Key modules

**`src/stream.ts`**: Public entry point exposing `stream()`, `agent()`, and `complete()`. Handles context normalization, tool validation, and provider dispatch.

**`src/assistant-stream.ts`**: Specialized `EventStream` for assistant responses. Completes on `done`/`error` events and exposes `result()` for the final message.

**`src/stream-controller.ts`**: Stateful builder that constructs `AssistantMessage` incrementally. Providers call `addPart()`, `delta()`, `endPart()`, etc.

**`src/event-stream.ts`**: Generic async-iterable queue implementing the producer-consumer pattern with a final result promise.

**`src/models.ts`**: Model registry with capabilities, pricing, and `calculateCost()`.

**`src/types.ts`**: All type definitions for messages, parts, events, and options.

**`src/providers/openai.ts`**: OpenAI Responses API adapter.

**`src/providers/anthropic.ts`**: Anthropic Messages API adapter.

**`src/providers/google.ts`**: Google Gemini API adapter.

**`src/usage.ts`**: Helper for creating empty `Usage` objects.

**`src/utils/json.ts`**: Safe JSON parsing for partial streaming data.

**`src/utils/sanitize.ts`**: Fixes invalid unicode surrogate pairs to prevent SDK failures.

**`src/utils/exhaustive.ts`**: Compile-time exhaustiveness checking helper for switch statements.

## Type system

The codebase uses discriminated unions extensively. Switch on the discriminator and use `exhaustive()` in the default case.

### Messages

Discriminated by `role`:

- **`system`** (`SystemMessage`): Contains `content: string`.
- **`user`** (`UserMessage`): Contains `content: string`.
- **`tool`** (`ToolMessage`): Contains `toolCallId`, `toolName`, `content`, and optional `isError`.
- **`assistant`** (`AssistantMessage`): Contains `provider`, `model`, `content: AssistantPart[]`, `stopReason`, and `usage`.

### AssistantPart

Discriminated by `type`:

- **`text`**: Standard text output with `text: string`.
- **`thinking`**: Reasoning/extended thinking content with `text: string`.
- **`tool_call`**: Tool invocation with `id`, `name`, and `args: unknown`.

All parts have optional `meta` for provider-specific round-trip data.

### AssistantStreamEvent

Discriminated by `type`:

- **`start`**: Stream began, provides initial draft.
- **`part_start`**: New part started at `index`.
- **`part_delta`**: Incremental text chunk for part at `index`.
- **`part_end`**: Part at `index` is complete.
- **`done`**: Stream finished with final `AssistantMessage`.
- **`error`**: Stream failed with partial message and `errorMessage`.

### AgentStreamEvent

For the `agent()` loop, discriminated by `type`:

- **`turn_start`**: New model turn beginning.
- **`assistant_event`**: Proxied `AssistantStreamEvent` from the current turn.
- **`tool_result`**: Tool execution completed with `ToolMessage`.
- **`done`**: Agent loop finished successfully.
- **`error`**: Agent loop failed.

## Streaming system

### EventStream

Generic async-iterable queue in `src/event-stream.ts`. The `push(event)` method enqueues an event, resolving immediately if a consumer is waiting. The `end(result?)` method signals completion and resolves all waiting consumers. Implements `[Symbol.asyncIterator]` to yield from the queue, blocking when empty. The `result()` method returns a promise that resolves when the stream completes.

### AssistantStream

Extends `EventStream<AssistantStreamEvent, AssistantMessage>`. Completes automatically on `type: "done"` or `type: "error"` events. The `resultOrThrow()` method throws if `stopReason` is `"error"` or `"aborted"`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [markusylisiurunen/iota](https://github.com/markusylisiurunen/iota) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
