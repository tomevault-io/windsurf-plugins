---
trigger: always_on
description: This file provides essential context for LLMs assisting with Logos Messaging development.
---

# AGENTS.md - AI Coding Context

This file provides essential context for LLMs assisting with Logos Messaging development.

## Project Identity

Logos Messaging is designed as a shared public network for generalized messaging, not application-specific infrastructure.

This project is a Nim implementation of a libp2p protocol suite for private, censorship-resistant P2P messaging. It targets resource-restricted devices and privacy-preserving communication.

Logos Messaging was formerly known as Waku. Waku-related terminology remains within the codebase for historical reasons.

### Design Philosophy

Key architectural decisions:

Resource-restricted first: Protocols differentiate between full nodes (relay) and light clients (filter, lightpush, store). Light clients can participate without maintaining full message history or relay capabilities. This explains the client/server split in protocol implementations.

Privacy through unlinkability: RLN (Rate Limiting Nullifier) provides DoS protection while preserving sender anonymity. Messages are routed through pubsub topics with automatic content-topic-based sharding (shard count is configurable; generation-zero defaults to 8 shards on cluster 0). Code prioritizes metadata privacy alongside content encryption.

Scalability via sharding: The network uses automatic content-topic-based sharding to distribute traffic. This is why you'll see sharding logic throughout the codebase and why pubsub topic selection is protocol-level, not application-level.

See [documentation](https://docs.waku.org/learn/) for architectural details.

### Core Protocols
- Relay: Pub/sub message routing using GossipSub
- Store: Historical message retrieval and persistence
- Filter: Lightweight message filtering for resource-restricted clients
- Lightpush: Lightweight message publishing for clients
- Peer Exchange: Peer discovery mechanism
- RLN Relay: Rate limiting nullifier for spam protection
- Metadata: Cluster and shard metadata exchange between peers
- Mix: Mixnet protocol for enhanced privacy through onion routing
- Rendezvous: Alternative peer discovery mechanism

### Key Terminology
- ENR (Ethereum Node Record): Node identity and capability advertisement
- Multiaddr: libp2p addressing format (e.g., `/ip4/127.0.0.1/tcp/60000/p2p/16Uiu2...`)
- PubsubTopic: Gossipsub topic for message routing (shard-based, e.g., `/waku/2/rs/<cluster-id>/<shard-id>`; the default is `/waku/2/rs/0/0`)
  - cluster-id: network id
  - shard-id: shard differentiator inside the network - drivers mesh forming.
    - autosharding: network supports n (configured) shards [0..n-1], shard derived from ContentTopic
- ContentTopic: Application-level message categorization (e.g., `/my-app/1/chat/proto`)
- Sharding: Partitioning network traffic across topics (static or auto-sharding)
- RLN (Rate Limiting Nullifier): Zero-knowledge proof system for spam prevention

### Specifications
All specs are at [rfc.vac.dev/waku](https://rfc.vac.dev/waku). RFCs use `WAKU2-XXX` format (not legacy `WAKU-XXX`).

## Architecture

### Protocol Module Pattern
Each protocol typically follows this structure:
```
waku_<protocol>/
├── protocol.nim       # Main protocol type and handler logic
├── client.nim         # Client-side API
├── rpc.nim           # RPC message types
├── rpc_codec.nim     # Protobuf encoding/decoding
├── common.nim        # Shared types and constants
└── protocol_metrics.nim  # Prometheus metrics
```

### WakuNode Architecture
- WakuNode (`waku/node/waku_node.nim`) is the central orchestrator
- Protocols are "mounted" onto the node's switch (libp2p component)
- PeerManager handles peer selection and connection management
- Switch provides libp2p transport, security, and multiplexing

Example protocol type definition:
```nim
type WakuFilter* = ref object of LPProtocol
  subscriptions*: FilterSubscriptions
  peerManager: PeerManager
  messageCache: TimedCache[string]
```

## Development Essentials

### Build Requirements
- Nim 2.x (check `logos_delivery.nimble` for minimum version)
- Rust toolchain (required for RLN dependencies)
- Build system: Make driven by Nimble (dependencies pinned in `nimble.lock`)

### Build System
The project uses a Makefile that drives Nimble. Dependencies are resolved from
`nimble.lock` into a local `nimbledeps/` directory (tracked by the
`NIMBLEDEPS_STAMP` target).
```bash
# Initial build (resolves Nimble deps automatically)
make wakunode2

# Build with custom flags
make wakunode2 NIMFLAGS="-d:chronicles_log_level=DEBUG"
```

Note: The build uses `--mm:refc` memory management (passed automatically by the Nimble tasks in `logos_delivery.nimble`). Only relevant if compiling outside the standard build system.

### Common Make Targets
```bash
make wakunode2          # Build main node binary
make test               # Run all tests
make testcommon         # Run common tests only
make libwaku            # Build the legacy C library (libwaku)
make liblogosdelivery. # Build actual C FFI library
make chat2              # Build chat example
make install-nph        # Install git hook for auto-formatting
```

### Testing
```bash
# Run all tests
make test

# Run specific test file
make test tests/test_waku_enr.nim

# Run specific test case from file

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [logos-messaging/logos-delivery](https://github.com/logos-messaging/logos-delivery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
