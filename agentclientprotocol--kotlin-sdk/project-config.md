---
trigger: always_on
description: This file is the repository-wide source of truth for agents changing the ACP Kotlin SDK. Keep it practical and current. When a core change alters the architecture, module boundaries, lifecycle rules, public API policy, or a shared implementation pattern, update this file in the same change.
---

# Repository Guide for Coding Agents

This file is the repository-wide source of truth for agents changing the ACP Kotlin SDK. Keep it practical and current. When a core change alters the architecture, module boundaries, lifecycle rules, public API policy, or a shared implementation pattern, update this file in the same change.

## Project Overview

This repository is a Kotlin Multiplatform implementation of the Agent Client Protocol (ACP). It provides:

- serializable ACP and JSON-RPC models;
- agent and client runtimes;
- protocol negotiation, request correlation, batching, and cancellation;
- STDIO and Ktor WebSocket transports;
- Ktor client/server integrations;
- shared integration tests and runnable samples.

Use the Gradle wrapper and JDK 21 for all builds. The shared convention plugin configures a JDK 21 toolchain, strict explicit API mode, and the supported Kotlin Multiplatform targets.

## Repository Map

### `:acp-model`

Pure protocol data and serialization:

- ACP request, response, notification, capability, and session models;
- JSON-RPC messages, request IDs, errors, and transport frames;
- stable models under `com.agentclientprotocol.model`;
- unstable protocol v2 models under `com.agentclientprotocol.model.v2`;
- stable-to-v2 conversions under `model.v2.conversion`;
- checked-in public API dump at `acp-model/api/acp-model.api`.

This module must not depend on runtime or transport implementations.

### `:acp`

The core runtime:

- `agent`: agent lifecycle, sessions, and client callbacks;
- `client`: client lifecycle, sessions, and update delivery;
- `common`: shared operations and session abstractions;
- `protocol`: JSON-RPC dispatch, correlation, batching, negotiation, and cancellation;
- `transport`: the transport contract and STDIO implementation;
- `util`: coroutine and pagination helpers.

The public API dump is `acp/api/acp.api`.

### `:acp-ktor`

Shared Ktor transport infrastructure, including `WebSocketTransport`. It builds on `:acp` and owns transport behavior that is common to Ktor clients and servers. Its public API dump is `acp-ktor/api/acp-ktor.api`.

### `:acp-ktor-client` and `:acp-ktor-server`

Client-side and server-side Ktor adapters. Keep shared WebSocket behavior in `:acp-ktor`; keep endpoint-specific setup in the corresponding adapter module. Their API dumps are in each module's `api` directory.

### `:acp-ktor-test`

Cross-transport integration and conformance-style tests. Use this module for behavior that should hold across STDIO/WebSocket or client/server wiring. It is a test module, not a published API module.

### `:samples:kotlin-acp-client-sample`

Runnable examples for stable ACP, direct v2 use, negotiation, and external-agent integration. Samples should demonstrate supported public APIs, not internal shortcuts.

### `buildSrc`

Shared Gradle convention and publishing plugins. `acp.multiplatform.gradle.kts` defines targets, explicit API mode, the JDK toolchain, and generated library-version constants. Changes here affect most modules and are core architectural changes.

## Architecture and Dependency Direction

The main runtime stack is:

```text
AgentSupport / AgentSession        Client support / ClientSessionOperations
             |                                      |
           Agent                                  Client
             \                                      /
                         Protocol
                            |
                         Transport
                    (STDIO or WebSocket)
```

Keep these boundaries intact:

- Models and JSON-RPC wire types belong in `:acp-model`.
- Request routing, correlation, negotiation, and cancellation belong in `Protocol`.
- Transport implementations move complete `TransportFrame` values and own transport state, I/O, and shutdown.
- Agent and client layers expose typed ACP behavior and session lifecycles. Do not leak raw transport concerns into their public APIs.
- Ktor-neutral behavior belongs in `:acp` or `:acp-model`; Ktor-specific behavior belongs in the `acp-ktor*` modules.
- Prefer `commonMain` for portable behavior. Put platform-specific code or tests in the narrowest applicable source set.

Avoid dependency cycles and upward dependencies. Lower layers must not depend on agent/client runtime types merely to simplify a call site.

## Protocol Versions and Unstable APIs

Stable ACP and protocol v2 coexist in this repository. Preserve their boundaries.

- Treat v2 APIs as unstable unless the protocol and repository explicitly promote them.
- Mark unstable declarations with `@UnstableApi`.
- Opt in explicitly at the narrowest useful scope with `@OptIn(UnstableApi::class)` or a file-level opt-in when most of a file is v2-specific.
- Keep version negotiation at the connection/runtime boundary. A connection speaks one negotiated protocol version.
- Put conversion logic in `model.v2.conversion`; do not scatter ad hoc stable/v2 conversions across runtimes and transports.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentclientprotocol/kotlin-sdk](https://github.com/agentclientprotocol/kotlin-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
