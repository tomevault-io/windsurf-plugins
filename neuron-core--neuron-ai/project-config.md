---
trigger: always_on
description: The protocol boundary between Neuron's native streamed objects and UI protocols such as AG-UI and the Vercel AI SDK data stream. `AgentChunkAdapter` is the same boundary for a destination that speaks no UI protocol.
---

# Stream Adapters

The protocol boundary between Neuron's native streamed objects and UI protocols such as AG-UI and the Vercel AI SDK data stream. `AgentChunkAdapter` is the same boundary for a destination that speaks no UI protocol.

## Ownership and layering

`StreamAdapterInterface`, `ProtocolEvent` and `SSEEncoder` live under `Workflow\Streaming` because the Workflow consumes the capability; the protocol-specific adapters live under `Agent\Adapters`. The Workflow-level contract must stay free of Agent- and memory-specific concepts so custom workflows can use it:

- a node may `yield` any object as live, intermediate output, and must still `return` its routing event;
- `Workflow::events()` exposes the native objects when no adapter is configured;
- the stream adapter, built for every segment by the `setStreamAdapter()` factory or the `streamAdapter()` hook, makes the Workflow convert output to `ProtocolEvent`s **once**: without a channel, pull consumers receive those events; with a channel, iteration also delivers them through `send(ProtocolEvent)`. `run()` consumes eagerly and returns state. A channel without an adapter receives only the segment lifecycle during iteration;
- `Agent::stream()` creates a streaming start request and always returns a lazy generator, including with a channel.

An adapter holds the state of one segment's stream, so its factory returns a new instance every time. A continuation's adapter starts from what the client already holds, which a protocol request carries: the AG-UI `messages`, the Vercel message and its `parts`. An adapter owns its protocol vocabulary and state, and nothing about bytes: `ProtocolEvent` carries a `type` and a JSON-serializable `data` payload (`jsonSerialize()` places the type first), and `SSEEncoder::encode()` / `frame()` turn events into `data:` lines at the HTTP edge, never inside the Workflow. The HTTP headers a protocol requires stay on the adapter (`getHeaders()`), because the Vercel header is part of that protocol's contract. Message IDs come from the chunks, which carry the ID of the message the provider returns (`src/Providers/AGENTS.md`); `UniqueIdGenerator` fills in only where nothing identifies the source.

## Contract

`start()` (optional framing), `transform(object $chunk)` (one object → zero or more `ProtocolEvent`s), and one terminal per segment, selected by the Workflow from the segment's outcome: `end()` on completion, `interrupt(InterruptRequest $request)` on suspension (the single current `InterruptRequest`, encoded so the client learns what the run waits for; return `[]` when the protocol cannot express a pause) and `error(Throwable $error)` on failure (return `[]` when the protocol has no failure frames). The Workflow calls `error()` itself for failures during streamed execution or chunk transformation; the run is settled as failed exactly as for a failing node, the channel still receives `failed()` and the exception is rethrown to the caller. After `error()` an adapter emits no further frames. An `InterruptEvent` never reaches `transform()`.

## Portable stream events

Nodes yield protocol-neutral UI information without importing AG-UI or Vercel types:

```text
workflow domain event → exact-class mapper (optional) → StreamEventInterface → AG-UI event | Vercel data part
```

`Events/` holds the portable value objects: `StepStartedStreamEvent`, `StepFinishedStreamEvent`, `ActivityStreamEvent` (a replaceable progress snapshot) and `CustomStreamEvent` (a named escape hatch with a JSON-serializable value). They are stream output, not workflow routing events, and contain no protocol field names; each adapter translates them (`STEP_STARTED` / `STEP_FINISHED` / `ACTIVITY_SNAPSHOT` / `CUSTOM` on AG-UI, transient `data-*` parts on Vercel).

`CustomizableStreamAdapterInterface::mapEvent()` (implemented once in `MapsStreamEvents`, used by every built-in adapter) lets a developer map their own domain events without touching the adapter. Register mappings where the factory builds the adapter:

```php
$adapter->mapEvent(
    IndexingProgress::class,
    static fn (IndexingProgress $event): ?ActivityStreamEvent => new ActivityStreamEvent(
        id: $event->jobId,
        type: 'indexing',
        data: ['processed' => $event->processed, 'total' => $event->total],
    ),
);
```

Resolution order: a yielded `StreamEventInterface` is encoded directly; then an **exact-class** mapping (no inheritance or first-match rules, so behavior stays predictable); then built-in native chunk conversion; anything else is ignored. A mapper returning `null` suppresses the event, and the implementation distinguishes "no mapping" from "mapped to null" so suppression never falls through to chunk handling. Mappers return portable value objects, never `ProtocolEvent`s or protocol arrays: the adapter is the single owner of the protocol vocabulary, and framing belongs to the transport edge. A `StreamEventInterface` the adapter cannot encode fails with an expressive exception rather than being dropped silently.

## Protocol invariants


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neuron-core/neuron-ai](https://github.com/neuron-core/neuron-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
