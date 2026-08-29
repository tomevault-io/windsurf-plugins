---
trigger: always_on
description: Build, architecture, and operational instructions for AI coding agents working on this codebase.
---

# TinyUa — Agent Context

Build, architecture, and operational instructions for AI coding agents working on this codebase.

## Build & Test

```bash
dotnet build TinyUa.sln
dotnet test TinyUa.Client.Tests/TinyUa.Client.Tests.csproj
dotnet test TinyUa.Security.Tests/TinyUa.Security.Tests.csproj
dotnet run --project TinyUa.Example -- selftest
```

`TinyUa.Explorer` and `TinyUa.CertGen` are Windows WPF applications. When their Windows SDK or
NuGet dependencies are unavailable, validate library changes by building/testing Core, Transport,
Client, Client.Tests, Security.Tests, and Benchmarks individually.

Live KEPServer checks default to `opc.tcp://localhost:49320`:

```bash
dotnet run --project TinyUa.Benchmarks -- none
dotnet run --project TinyUa.Benchmarks -- substress
dotnet run --project TinyUa.Benchmarks -- memory
```

`memory` runs continuously until Ctrl+C. It batch-reads verified-writable variables below
`Objects/DemoCH/Data`, writes their just-read values back, and reports allocation, GC, ArrayPool,
and encoder-pool telemetry.

## Architecture

```
TinyUa.Core  (no external dependencies, net8.0)
  +-- TinyUa.CertGen  (WPF, net8.0-windows)
  +-- TinyUa.Transport  (references Core)
        +-- TinyUa.Client  (references Transport)
              +-- TinyUa.Example  (Console, + OPCF SDK for in-process server)
              +-- TinyUa.Explorer  (WPF, net8.0-windows, + MVVM/WPF-UI)
              +-- TinyUa.Benchmarks  (+ BenchmarkDotNet)
              +-- TinyUa.Client.Tests  (xUnit)
              +-- TinyUa.Security.Tests  (xUnit, + OPCF SDK)
```

Core, Transport, and Client have zero dependency on the OPC Foundation SDK. Binary encoding,
transport, security, sessions, services, subscriptions, and reconnection are implemented from
scratch using .NET 8 and `System.Security.Cryptography`.

The OPCF SDK (`OPCFoundation.NetStandard.Opc.Ua` v1.5.374.126) is used only by Example and
Security.Tests to host an in-process reference server.

## Key Entry Points

- [TinyUa.Client/UaClient.cs](TinyUa.Client/UaClient.cs) — Public client API, lifecycle events,
  Read/Write/Browse/Subscribe helpers, and the single reconnect entry point
- [TinyUa.Client/ClientBuilder.cs](TinyUa.Client/ClientBuilder.cs) — Fluent builder
- [TinyUa.Client/UaClientOptions.cs](TinyUa.Client/UaClientOptions.cs) — Client, reconnect,
  keep-alive, subscription-dispatch, identity, certificate, and security options
- [TinyUa.Client/ClientStateMachine.cs](TinyUa.Client/ClientStateMachine.cs) — Idempotent lifecycle
  state transitions; `Replay` is the explicit repeated-notification path
- [TinyUa.Client/Connection/ReconnectEngine.cs](TinyUa.Client/Connection/ReconnectEngine.cs) —
  Single-flight reconnect, session recovery, Republish, and subscription rebuild
- [TinyUa.Client/Connection/UaSocketClient.cs](TinyUa.Client/Connection/UaSocketClient.cs) —
  Buffered socket receive ring, request correlation, gather send, and connection-loss signal
- [TinyUa.Client/Services/ReadService.cs](TinyUa.Client/Services/ReadService.cs) — `ReadResult`,
  `ReadValueId`, Read request/response
- [TinyUa.Client/Services/WriteService.cs](TinyUa.Client/Services/WriteService.cs) — `WriteResult`,
  `WriteValue`, Write request/response
- [TinyUa.Client/Subscriptions/PublishEngine.cs](TinyUa.Client/Subscriptions/PublishEngine.cs) —
  Session-level finite Publish-request credit window
- [TinyUa.Client/Subscriptions/SubscriptionManager.cs](TinyUa.Client/Subscriptions/SubscriptionManager.cs) —
  `DataChangeHandler`, `DataChangeHandlerEx`, `Subscription`, ordered callback worker, queue metrics
- [TinyUa.Client/Subscriptions/SubscriptionDispatchOptions.cs](TinyUa.Client/Subscriptions/SubscriptionDispatchOptions.cs) —
  Queue capacity and `Wait`/`DropOldest`/`DropNewest` policy
- [TinyUa.Core/Binary/BufferLease.cs](TinyUa.Core/Binary/BufferLease.cs) — Class-based ArrayPool
  ownership, sensitive-buffer clearing, large-buffer bypass, and pool telemetry
- [TinyUa.Core/Binary/BinaryEncoderPool.cs](TinyUa.Core/Binary/BinaryEncoderPool.cs) — Bounded encoder cache
- [TinyUa.Transport/MessageChunk.cs](TinyUa.Transport/MessageChunk.cs) — OPC UA Part 6 chunking,
  segmented signing, encryption, and pooled wire buffers
- [TinyUa.Transport/SecureConnection.cs](TinyUa.Transport/SecureConnection.cs) — Secure-channel token
  rotation and message framing
- [TinyUa.Core/Security/SecurityPolicyFactory.cs](TinyUa.Core/Security/SecurityPolicyFactory.cs) —
  Security policy normalization and construction
- [TinyUa.Core/Types/Variant.cs](TinyUa.Core/Types/Variant.cs) — Variant inference and explicit conversion
- [TinyUa.Core/Types/UaTypes.cs](TinyUa.Core/Types/UaTypes.cs) — `StatusCode`, `DataValue`,
  `QualifiedName`, and `LocalizedText`
- [TinyUa.Benchmarks/MemoryKepwareBenchmark.cs](TinyUa.Benchmarks/MemoryKepwareBenchmark.cs) —
  Continuous batch Read/Write memory and pool monitor
- [TinyUa.Example/Program.cs](TinyUa.Example/Program.cs) — Runnable examples and self-test

## Engineering Conventions

- **Public client namespaces**: use `TinyUa.Client`, `TinyUa.Client.Services`,
  `TinyUa.Client.Subscriptions`, and `TinyUa.Client.Discovery`. Do not use the removed
  `TinyUa.Core.Client*` namespaces.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [renrouputao/tinyua](https://github.com/renrouputao/tinyua) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
