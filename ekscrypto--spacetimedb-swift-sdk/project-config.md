---
trigger: always_on
description: This is a Swift SDK for SpacetimeDB, an integrated API and database system. The SDK enables Swift applications to connect to SpacetimeDB servers, subscribe to data changes, and execute server-side reducers.
---

# Claude Code Assistant Instructions

## Project Overview
This is a Swift SDK for SpacetimeDB, an integrated API and database system. The SDK enables Swift applications to connect to SpacetimeDB servers, subscribe to data changes, and execute server-side reducers.

## Important Context for AI Assistants

### Current Status
⚠️ **IMPORTANT**: Before making any changes, read the [README.md](README.md) file for:
- Current SDK status and maturity level
- Complete list of implemented/unimplemented features
- Known limitations and roadmap items
- Test coverage details

This document focuses on **implementation details** not covered in the README.

### Key Technical Concepts

1. **BSATN (Binary Spacetime Algebraic Type Notation)**: Binary serialization format for data exchange
   - All data types must be encoded/decoded using BSATN
   - Implementation in `Sources/BSATN/` directory
   - Large integers (UInt128, UInt256, Int128, Int256) use hex string encoding in JSON

2. **WebSocket Protocol**: All communication uses binary WebSocket messages
   - Messages have a type byte followed by BSATN-encoded payload
   - Client → Server and Server → Client message types are defined in `Sources/SpacetimeDB/Tags.swift`

3. **Table Row Decoders**: Tables require registered decoders before data can be received
   - Adopt the `BSATNRow` protocol on the row struct (just `init(reader:)` + `static var tableName`); register with `client.registerTableRowDecoder(MyRow.self)`. For tables with a primary key, adopt `BSATNTableWithPrimaryKey` instead — that unlocks `.updated(old:new:)` events on the per-row stream.
   - Hand-written decoders may conform to `TableRowDecoder` directly with a `ProductModel` and a `decode(modelValues:)` method. The default `decode(reader:)` extension reads the AlgebraicValue first and dispatches.
   - The codegen tool `spacetime-swift generate` emits `BSATNRow`/`BSATNTableWithPrimaryKey`-based files automatically.

4. **Reducers**: Server-side functions that modify database state
   - Must implement `Reducer` protocol with BSATN argument encoding
   - Called via `client.callReducer(reducer)`
   - Codegen also emits per-reducer `<Name>Reducer` structs.

5. **Subscription Management**: Clients can subscribe and unsubscribe from data changes
   - `let handle = try await client.subscribe([...])` returns a `SubscriptionHandle`; await `handle.applied()` and later `handle.unsubscribe()`. Use `client.subscribeToAllTables()` to subscribe to every registered table.
   - Wire-level primitives `subscribeMulti(queries:queryId:)` / `subscribe(queries:requestId:)` / `unsubscribe(queryId:)` / `unsubscribeSingle(queryId:)` are also exposed for callers that need to manage query IDs themselves.

6. **Event surface**: AsyncStreams or `SpacetimeDBClientDelegate` callbacks — both fan out from the same receive loop.
   - `client.connectionEvents` — `.connected/.reconnecting/.disconnected/.error`
   - `client.reducerEvents` — typed `ReducerStatus` + `EnergyQuanta`
   - `client.subscriptionEvents` — `.applied/.unsubscribed/.error`
   - `client.tableEvents(named:)` — batched per-table updates
   - `client.rowEvents(table:)` — per-row events; PK-matched delete+insert pairs collapse to `.updated(old:new:)` automatically
   - `connect(delegate:)` is optional — pass `nil` to use only the streams.

### Architecture Decisions

- **No SDK-level interpretation**: The SDK passes data to the delegate without interpreting changes — rename detection is done by the client. The streams API does it for free via PK matching when the row adopts `BSATNTableWithPrimaryKey`.
- **Batched updates**: All table updates for a transaction are batched before notifying delegate AND before fanning out to the per-table stream.
- **Offsets in BsatnRowList**: Offsets mark the START position of rows in the data blob, not the end
- **Subscription readiness**: Wait for `SubscriptionHandle.applied()` (or the `onSubscribeMultiApplied` delegate call) before processing user commands. **Streams-mode trap**: subscribing before the server's `IdentityToken` arrives hangs against maincloud — always wait for `ConnectionEvent.connected` before calling `subscribe(...)`. See `Sources/quickstart-chat/StreamsChat.swift` for the canonical pattern.

### Testing Commands
When implementing new features, test with the quickstart-chat application:
```bash
swift build
./.build/debug/quickstart-chat
```

**⚠️ IMPORTANT Testing Notes:**
1. **Always test with the live client** after making changes to ensure the implementation still works with the actual SpacetimeDB server
2. **The client is interactive** - it waits for user input. Enter `/quit` to exit properly. If you just run it without input, it will appear to "hang" but it's actually waiting for commands
3. **Use echo for automated testing**: `echo "/quit" | swift run quickstart-chat` to automatically exit after connection
4. **Check connection success**: The client should show "✅ Connected to SpacetimeDB!" and receive user/message data
5. **Verify with real operations**: Test actual commands like `/name TestUser`, `/sub`, `/unsub`, and sending messages to ensure protocol changes work

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ekscrypto/spacetimedb-swift-sdk](https://github.com/ekscrypto/spacetimedb-swift-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
