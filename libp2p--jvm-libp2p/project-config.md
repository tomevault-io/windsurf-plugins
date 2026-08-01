---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

jvm-libp2p is a JVM implementation of the [libp2p](https://libp2p.io/) networking stack, written in Kotlin. It provides peer-to-peer networking capabilities including transport protocols (TCP, QUIC, WebSocket), security channels (Noise, TLS), stream multiplexing (Yamux, Mplex), and pub/sub messaging (Gossipsub, Floodsub).

Notable users: Teku (Ethereum Consensus Layer client), Nabu (minimal IPFS), Peergos (peer-to-peer encrypted filesystem).

## Build Commands

```bash
# Build the entire project
./gradlew build

# Run all tests (excludes interop tests tagged with "interop")
./gradlew test

# Run tests for a specific module
./gradlew :libp2p:test

# Run a specific test class
./gradlew :libp2p:test --tests "io.libp2p.pubsub.gossip.GossipRpcPartsQueueTest"

# Run a specific test method
./gradlew :libp2p:test --tests "io.libp2p.pubsub.gossip.GossipRpcPartsQueueTest.mergeMessageParts*"

# Check code formatting
./gradlew spotlessCheck

# Apply code formatting
./gradlew spotlessApply

# Run static analysis (Detekt)
./gradlew detekt

# Generate documentation
./gradlew dokkaHtml
# Output in build/dokka/

# Clean build artifacts
./gradlew clean
```

**Requirements:** JDK 11 or higher

**Module Structure:**
- `:libp2p` - Main library module
- `:tools:simulator` - Gossip network simulator
- `:tools:schedulers` - Test scheduling utilities
- `:examples:chatter`, `:examples:cli-chatter`, `:examples:pinger` - Example applications
- `:interop-test-client` - Interoperability testing client

## Architecture Overview

### Core Abstraction Layers

The library follows a layered architecture with protocol negotiation at each layer:

```
Application Layer
    ↓ (Protocol negotiation via multistream-select)
Stream/Protocol Layer (PingProtocol, ChatProtocol, PubsubRouter)
    ↓ (Stream creation)
Stream Multiplexing Layer (Yamux, Mplex)
    ↓ (Multiplexer negotiation)
Security Layer (Noise, TLS)
    ↓ (Security negotiation)
Transport Layer (TCP, QUIC, WebSocket)
    ↓
Raw Network
```

### Key Interfaces and Their Roles

**`Host`** (`core/Host.kt`):
- Main entry point for all libp2p operations
- Manages identity (`PeerId`, `PrivKey`), network, and protocol handlers
- Created via DSL builder: `host { identity { ... }; transports { ... }; protocols { ... } }`

**`Network`** (`core/Network.kt`):
- Manages transports and active connections
- Handles `listen()` and `dial()` operations
- Reuses connections to the same peer

**`Connection`** and **`Stream`** (both extend `P2PChannel`):
- `Connection`: Secured, multiplexed connection between two peers
- `Stream`: Logical stream over a connection for a specific protocol

**`Transport`** (`transport/Transport.kt`):
- Handles raw connection establishment (TCP, QUIC, WebSocket)
- Each transport parses specific multiaddr formats (e.g., `/ip4/127.0.0.1/tcp/30333`)

**`SecureChannel`** (`security/SecureChannel.kt`):
- Protocol binding for security layer negotiation
- Returns `SecureChannel.Session` with `remoteId`, `remotePubKey`
- Implementations: `NoiseXXSecureChannel` (production), `TlsSecureChannel` (beta)

**`StreamMuxer`** (`mux/StreamMuxer.kt`):
- Protocol binding for multiplexer negotiation
- Returns `StreamMuxer.Session` for creating/receiving streams
- Implementations: `MplexStreamMuxer` (production), `YamuxStreamMuxer` (beta)

### The Connection Upgrade Pipeline

When a raw transport connection is established, it goes through staged upgrades:

```
1. Raw Transport (TCP/QUIC/WS)
   ↓
2. ConnectionBuilder (transport/implementation/ConnectionBuilder.kt)
   ↓
3. Security Negotiation → SecureChannel.Session
   ↓
4. Multiplexer Negotiation → StreamMuxer.Session
   ↓
5. Full Connection Ready → ConnectionOverNetty
```

**Key Class:** `ConnectionUpgrader` (`transport/implementation/ConnectionUpgrader.kt`)
- Orchestrates security and muxer protocol negotiation
- Uses `MultistreamProtocol` for protocol selection
- Supports early muxer negotiation (TLS 1.3 feature)

### Protocol Handler Pattern

Custom protocols implement `ProtocolHandler<TController>`:

```kotlin
// Define protocol binding
StrictProtocolBinding("/ipfs/ping/1.0.0", PingProtocol())

// Implement handler
class PingProtocol : ProtocolHandler<PingController> {
    override fun onStartInitiator(stream: Stream): CompletableFuture<PingController>
    override fun onStartResponder(stream: Stream): CompletableFuture<PingController>
}
```

See `examples/chatter/ChatProtocol.kt` for a complete example.

### Pub/Sub Architecture

The pub/sub system is located in `pubsub/` and follows this structure:

**`AbstractRouter`** (`pubsub/AbstractRouter.kt`):
- Base class providing common pubsub logic
- Manages peer subscriptions via `peersTopics` (multi-bimap)
- Implements message validation, deduplication (via `SeenCache`), and batching
- Uses single-threaded event loop (`P2PService`) for thread-safety

**Message Batching via `RpcPartsQueue`**:
- Per-peer queue that accumulates message parts before transmission
- Pattern: accumulate parts → flush via `takeMerged()` → send merged RPC
- Default implementation merges all parts into single RPC

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [libp2p/jvm-libp2p](https://github.com/libp2p/jvm-libp2p) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
