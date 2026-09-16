---
trigger: always_on
description: Repository: `cascade-realtime-gateway` · Module: `github.com/rasonyang/cascade-realtime-gateway` · Binary: `cascade`
---

# Cascade

Repository: `cascade-realtime-gateway` · Module: `github.com/rasonyang/cascade-realtime-gateway` · Binary: `cascade`

OpenAI Realtime **GA**-compatible voice gateway. Speaks the OpenAI Realtime WebSocket protocol externally; runs a Go-native session engine internally with a cascaded ASR → LLM → TTS pipeline. The compatibility target is the GA compatibility profile defined in `docs/protocol-profile.md`, not full parity.

## Language

English is the default for everything — code, identifiers, comments, docs, commit messages, log messages, and conversation with the user — unless explicitly told otherwise.

## Stack

Go, net/http, WebSocket, slog, OpenTelemetry. Single binary, single config file, zero external dependencies. Databases, Redis, Kafka, microservices: not allowed without a proven need.

## Architecture

```
Admin REST /admin/v1 → RuntimeConfig (atomic snapshot) → resolved per new connection
                                  ↓
WebSocket
  ↓
Protocol Adapter   ← protocol state: event/item/response IDs, serialization, profile validation, truncate conversion
  ↓ internal Command / Event (Go types)
Session (actor)    ← session state: Conversation, InputAudioBuffer, TurnManager, Response lifecycle
  ↓ spawned per generation
Response Pipeline  ← one goroutine group per response: LLM → sentencer → TTS, discarded when done
  ↓
Providers (ASR / LLM / TTS interfaces)
```

Iron rules:

- The OpenAI protocol must not become the internal domain model. Internal state is authoritative; protocol events are a projection. `internal/session` must not import the protocol or admin packages; `internal/provider` must not import session, protocol or admin; `internal/admin` must not import session, protocol or server. Only `internal/server` and `cmd/cascade` import admin.
- Only one protocol version is targeted: GA (`session.type="realtime"`, `output_modalities`, `session.audio.input/output`, `response.output_audio.delta`, etc.). Beta shapes are not supported. Event names, fields, ordering, and lifecycle follow the official docs — never guess. Unsupported fields/events are rejected with an `error` event by default, never silently ignored.
- Protocol conformance testing has two layers: deterministic control sequences are checked by full-order golden-trace comparison; non-deterministically interleaved streams (audio delta / transcript delta) are checked only against causal invariants — in-order within each stream, `created` before any delta, every stream closed by `done`, no new delta after `cancelled`.

## Concurrency model

- The Session is a single-goroutine actor that owns all mutable state. Callers post Commands; the actor emits Events. No locks, no shared state.
- Command (into the actor, expresses intent) and Event (out of the actor, states a fact) are two separate type sets; never mixed.
- Data plane uses typed channels; control plane is out-of-band (context cancellation + actor commands) and never queues behind back-pressured audio.
- Generation stamps apply only to events produced by pipeline goroutines (deltas, progress facts); the actor filters stale generations at ingress. Response terminal events (`response.done`, etc.) are emitted synchronously by the actor on FSM transition, carry no generation, and are never filtered. Inbound audio and control commands are outside generation filtering.
- Interrupt order is fixed: cancel ctx → mark FSM cancelled → actor emits terminal events synchronously → bump generation. Cancellation must propagate immediately and close provider streams; no stream may keep burning tokens. Generation filtering is only a race backstop, not a substitute for prompt cleanup.
- All WebSocket writes go through a single writer goroutine.

## Input audio

- Transport layer: the WS read loop never blocks; audio enters a bounded queue; a full queue is an explicit error and disconnect, never a silent drop.
- Domain layer: `InputAudioBuffer` holds uncommitted audio and owns append / clear / commit slicing, prefix-padding rollback, `audio_start_ms` / `audio_end_ms` computation, and the memory cap. All millisecond values come from it; VAD only reports relative offsets.

## Turns and interruption

- TurnManager is a pure state machine called by the actor: it takes VAD/ASR facts and client commands, and returns EmitSpeech / Interrupt / Commit / Trigger decisions. It starts no goroutines and never touches the conversation.
- turn_detection modes: `server_vad` / `semantic_vad` / `null`. Under `null`, VAD does not run, no speech events are emitted, and commit and response creation are entirely client-driven.
- In VAD modes, `create_response` and `interrupt_response` are orthogonal: `create_response=false` disables only auto-Trigger — **Commit still happens** and `input_audio_buffer.committed` is still emitted; `interrupt_response=false` disables only auto-interrupt — speech events are still emitted.
- Speech-start detection (the interrupt trigger) uses the engine's own acoustic VAD ahead of ASR and does not depend on the ASR vendor. Under `semantic_vad`, end-of-turn consumes ASR transcripts; this is an approximation and is recorded in `docs/decisions.md`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rasonyang/cascade-realtime-gateway](https://github.com/rasonyang/cascade-realtime-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
