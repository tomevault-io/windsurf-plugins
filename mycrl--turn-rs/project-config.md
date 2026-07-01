---
trigger: always_on
description: This document targets two audiences:
---

# AGENTS.md

This document targets two audiences:

1) LLMs/agents: quickly understand project structure, entry points, run flow, configuration, and risks.
2) Human developers/operators: follow the steps to build, configure, start, and verify the service.

## Project Overview

turn-rs is a TURN/STUN server implemented in Rust for WebRTC NAT traversal and media relay. It focuses on high performance and low configuration cost, and provides optional gRPC management APIs, Prometheus metrics, and Hook callbacks.

## Core Capabilities

- TURN/STUN protocol support with TCP/UDP transport.
- Long-term credential mechanism with static users and hook-based dynamic auth.
- Optional gRPC management API and Prometheus metrics exporter.
- Multi-interface listeners and external address announcement.

## Key Entry Points and Directories

- Runtime entry: [src/main.rs](src/main.rs)
- Library entry: [src/lib.rs](src/lib.rs)
- Service and session logic: [src/service](src/service)
- Protocol handling and codecs: [src/codec](src/codec)
- Transport providers (UDP/TCP): [src/server/provider](src/server/provider)
- Config and logging: [src/config.rs](src/config.rs), [src/logger.rs](src/logger.rs)
- gRPC API and client SDK: [sdk/protos/server.proto](sdk/protos/server.proto), [sdk/src/lib.rs](sdk/src/lib.rs)
- Sample config: [turn-server.toml](turn-server.toml)
- Docs entry: [docs/README.md](docs/README.md)

## Source Code Architecture (Modules and Responsibilities)

This section explains how the server is organized internally and how the main data flow works.

### High-level runtime flow

1) [src/main.rs](src/main.rs) loads config, initializes logging, and builds the Tokio runtime.

2) [src/lib.rs](src/lib.rs) `start_server()` constructs `Statistics`, a `Handler`, and a `Service`, then spawns:

- transport servers (UDP/TCP) via [src/server](src/server)
- optional Prometheus exporter via [src/prometheus.rs](src/prometheus.rs)
- optional gRPC API via [src/api.rs](src/api.rs)

### Core modules

1) [src/service](src/service): TURN service core, shared state, and routing glue.

- `Service` holds realm, interfaces, session manager, and handler, and creates per-connection routers.
- `ServiceHandler` defines the hooks the protocol layer uses for auth and lifecycle callbacks.
- [src/service/routing.rs](src/service/routing.rs) parses STUN/TURN messages and dispatches by method.

2) [src/service/session](src/service/session): Session state, allocation, permissions, and channel bindings.

- `Identifier` (source + interface) is the primary session key.
- `SessionManager` owns sessions, port mappings, permissions, and channel relay tables.
- `Session` tracks authentication state, nonce, allocated port, channels, permissions, and expiry.
- [src/service/session/ports.rs](src/service/session/ports.rs) provides `PortAllocator` and `PortRange`.

3) [src/server](src/server): Transport orchestration and cross-protocol forwarding.

- [src/server/mod.rs](src/server/mod.rs) `start_server()` spawns TCP/UDP listeners per configured interface and aborts all servers if any one exits.
- [src/server/switch.rs](src/server/switch.rs) `Switch` maps each session `Identifier` to an internal channel for forwarding relayed packets between sockets; missing destinations are dropped silently.
- [src/server/buffer.rs](src/server/buffer.rs) provides a global memory pool (`Buffer`) backed by a lock-free `crossbeam_queue::ArrayQueue` with a background task that shrinks idle buffers to avoid leaks.

4) [src/server/provider](src/server/provider): Transport abstraction and server loop.

- [src/server/provider/mod.rs](src/server/provider/mod.rs) defines the `ProviderServer`/`ProviderStream` traits and `ServerOptions`. `ProviderServer::start` binds sockets, spawns per-connection tasks, routes packets, applies TCP channel-data padding, drives stats reporting, and handles idle timeout.
- [src/server/provider/udp.rs](src/server/provider/udp.rs) implements `UdpServer`/`UdpSession` (a single shared socket demultiplexed into per-peer channels).
- [src/server/provider/tcp.rs](src/server/provider/tcp.rs) implements `TcpServer` with an optional TLS (`MaybeSslStream`) accept path.

5) [src/handler.rs](src/handler.rs): Implements `ServiceHandler`.

- Auth flow: static credentials -> static auth secret -> optional Hook `GetPassword`.
- Lifecycle events: allocation, channel bind, permission create, refresh, destroy (sent to Hook service when enabled).

6) [src/api.rs](src/api.rs): gRPC management API and Hook client implementation.

- `TurnService` exposes GetInfo/GetSession/GetSessionStatistics/DestroySession.
- `RpcHooksService` maintains a client + buffered event channel to the external Hook service.
- The protobuf definitions and generated types now live in the `turn-server-sdk` crate; this module consumes them via `sdk::protos::*`.

7) [src/codec](src/codec): STUN/TURN codec and crypto.

- Decoder differentiates STUN messages vs. ChannelData.
- Message encoder/decoder handles attributes, integrity, and fingerprint.
- `crypto` contains HMAC and password derivation helpers.

8) [src/statistics.rs](src/statistics.rs): Per-session counters and reporting.

- `StatisticsReporter` aggregates per-session bytes/packets and error counts.
- Integrates with Prometheus metrics when enabled.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mycrl/turn-rs](https://github.com/mycrl/turn-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
