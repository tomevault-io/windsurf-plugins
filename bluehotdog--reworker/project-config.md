---
trigger: always_on
description: **@bluehotdog/reworker** is a ReScript library providing type-safe, chunked message passing for WebWorkers, ServiceWorkers, and browser extensions using GADTs (Generalized Algebraic Data Types).
---

# ReWorker - Universal Message Passing Library

**@bluehotdog/reworker** is a ReScript library providing type-safe, chunked message passing for WebWorkers, ServiceWorkers, and browser extensions using GADTs (Generalized Algebraic Data Types).

## Core Features

### 1. Type-Safe Message Passing with GADTs
- **GADT Definition**: `type rec message<_> = ..` in `src/Types.res` enables fully typed request-response pairs
- **Type Safety**: Each message type statically defines its expected response type
- **Extensible**: Messages can be extended across packages using `type Types.message<_> += ...`

### 2. Chunked Message Support
- **Unlimited Size**: Automatically chunks large messages to bypass Chrome extension size limits
- **Transparent**: Chunking happens automatically when messages exceed threshold
- **Reassembly**: Automatic message reconstruction on the receiving end

## Architecture Overview

```
src/
├── Types.res              # GADT message type definitions
├── Runtime.res            # Main messaging runtime with functor-based bindings
├── TransportMessage.res   # Internal transport layer for chunking
├── RequestHandler.res     # Chunk reassembly and message forwarding
├── MessageChunker.res     # Core chunking functionality with internal Chunk module
├── Response.res           # Generic response types for Chrome extension patterns
└── Id.res                # Opaque UUID generation for chunk tracking
```

## Development Agent Specializations

### GADT Message System Agent
**Use this agent for**: Defining new message types, type safety issues, extending message variants

**Key Concepts**:
- GADTs allow each message constructor to specify its response type
- Example: `| GetUser(string): Types.message<User.t>`
- Response type is encoded in the message type itself
- Enables compile-time verification of request-response pairs

**Common Tasks**:
- Add new message variants with proper response types
- Fix GADT type inference issues
- Extend message types across packages
- Debug message type mismatches

### Transport Layer Agent
**Use this agent for**: Internal chunking implementation, transport message processing, chunk reassembly

**CRITICAL: TransportMessage is Completely Internal**:
- **Users never see TransportMessage.t** - it's purely an internal implementation detail
- **Native bindings work with raw ReScript values** - no knowledge of transport types needed
- **Runtime.Make handles all conversion** - user types ↔ transport types ↔ native bindings

**How Transport Chunking Works**:
- Messages over threshold are automatically chunked by `Runtime.Make` functor
- **Internal-only**: Transport layer uses `TransportMessage.t<'response>` types separate from user messages
- Three transport message variants (all internal):
  - `UserMessage(message)` - Direct small messages
  - `IntermediateChunk(chunk): t<chunkAck>` - Mid-transfer chunks expecting acknowledgment
  - `FinalChunk(chunk): t<'response>` - Last chunk expecting original response type
- **User experience**: Send `GetUser("123")`, receive `User.t` - chunking is invisible

**Transport Flow Architecture**:
```
User Level:                     Transport Level:                 Receiving Side:
┌─────────────────┐            ┌─────────────────┐              ┌─────────────────┐
│ GetUser("123")  │  Runtime   │ Check size      │              │ RequestHandler  │
│                 │ ────────▶  │ >31MB? Chunk    │              │ (Reassembly)    │
└─────────────────┘            └─────────┬───────┘              └─────────┬───────┘
                                         │                                │
                                         ▼                                ▼
                               ┌─────────────────┐              ┌─────────────────┐
                               │ IntermediateChunk│ JSON        │ Collect chunk   │
                               │ chunk #1        │────────────▶│ Ack: ChunkAck   │
                               └─────────────────┘              └─────────────────┘
                                         │                                │
                                         ▼                                ▼
                               ┌─────────────────┐              ┌─────────────────┐
                               │ IntermediateChunk│ JSON        │ Collect chunk   │
                               │ chunk #2        │────────────▶│ Ack: ChunkAck   │
                               └─────────────────┘              └─────────────────┘
                                         │                                │
                                         ▼                                ▼
                               ┌─────────────────┐              ┌─────────────────┐
                               │ FinalChunk      │ JSON        │ Reassemble      │
                               │ chunk #3        │────────────▶│ Call user handler│
                               └─────────────────┘              │ Return: User.t  │
                                                                └─────────────────┘
```

**Key Files**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BlueHotDog/reworker](https://github.com/BlueHotDog/reworker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
