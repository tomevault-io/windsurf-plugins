---
trigger: always_on
description: This file provides guidance to Claude Code when working with this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## What is SharpStreamer

SharpStreamer is a .NET 10 library implementing the **Transactional Outbox/Inbox Pattern** for reliable, ordered event communication between microservices. Events are inserted into the outbox table within the caller's DB transaction, then background services pick them up, publish via a transport, store in the inbox, and process via MediatR handlers.

**Supported combinations:**
- **Transports (message brokers):** RabbitMQ, Kafka, PostgreSQL (loopback), SQLite (loopback)
- **Storage backends:** PostgreSQL (Npgsql), SQLite

All projects target `net10.0`. The recommended production combination is **RabbitMQ transport + Npgsql storage**.

## Build & Test Commands

```bash
# Build entire solution
dotnet build
dotnet build --configuration Release

# Run Npgsql tests (requires Docker — uses Testcontainers for PostgreSQL)
dotnet test tests/Storage.Npgsql.Tests/Storage.Npgsql.Tests.csproj

# Run SQLite tests (no Docker required — uses temp file DB)
dotnet test tests/Storage.Sqlite.Tests/Storage.Sqlite.Tests.csproj

# Run a single test by name
dotnet test tests/Storage.Sqlite.Tests/Storage.Sqlite.Tests.csproj --filter "FullyQualifiedName~MethodName"
```

## Publishing NuGet Packages

The helper script `PublishProjectScript.cs` (top-level statements, `net10.0`) auto-reads `<Version>` from the `.csproj`, packs in Release mode, and pushes:

```bash
dotnet run .\PublishProjectScript.cs -- <PROJECT_NAME> <NUGET_API_KEY>
# Example:
dotnet run .\PublishProjectScript.cs -- DotNetCore.SharpStreamer.Transport.RabbitMq YOUR_KEY
```

**Remember to bump `<Version>` in the target `.csproj` before publishing.**

## Project Layout

```
src/
  DotNetCore.SharpStreamer/                    # Core: interfaces, entities, attributes, DI, caching
  DotNetCore.SharpStreamer.Storage.Npgsql/     # EF Core + PostgreSQL storage (schema: sharp_streamer)
  DotNetCore.SharpStreamer.Storage.Sqlite/     # EF Core + SQLite storage (no schema prefix)
  DotNetCore.SharpStreamer.Transport.Npgsql/   # PostgreSQL loopback transport
  DotNetCore.SharpStreamer.Transport.Sqlite/   # SQLite loopback transport
  DotNetCore.SharpStreamer.Transport.Kafka/    # Kafka transport
  DotNetCore.SharpStreamer.Transport.RabbitMq/ # RabbitMQ transport
samples/
  DotNetCore.SharpStreamer.RabbitMq.Npgsql/    # RabbitMQ + Npgsql sample
  DotNetCore.SharpStreamer.Kafka.Npgsql/       # Kafka + Npgsql sample
  DotNetCore.SharpStreamer.Npgsql.Npgsql/      # Npgsql loopback sample
  DotNetCore.SharpStreamer.Sqlite.Sqlite/      # SQLite loopback sample
tests/
  Storage.Npgsql.Tests/                         # Integration tests (Testcontainers + PostgreSQL)
  Storage.Sqlite.Tests/                         # Integration tests (temp file SQLite)
```

## Current Package Versions

| Package | Version |
|---|---|
| DotNetCore.SharpStreamer (Core) | 4.0.2 |
| Storage.Npgsql | 4.0.1 |
| Storage.Sqlite | 1.0.1 |
| Transport.RabbitMq | 3.0.3 |
| Transport.Kafka | (no version tag) |
| Transport.Npgsql | 4.0.1 |
| Transport.Sqlite | 1.0.1 |

## Core Architecture

### DI Registration Order (Critical)

MediatR **must** be registered before SharpStreamer, then storage, then transport:

```csharp
builder.Services.AddMediatR(options =>
{
    options.RegisterServicesFromAssemblies(Assembly.GetExecutingAssembly());
    options.Lifetime = ServiceLifetime.Transient;
});

builder.Services
    .AddSharpStreamer("SharpStreamerSettings", Assembly.GetExecutingAssembly())
    .AddSharpStreamerStorageNpgsql<YourDbContext>()   // or AddSharpStreamerStorageSqlite<T>()
    .AddSharpStreamerTransportRabbitMq();             // or Kafka/Npgsql/Sqlite
```

`AddSharpStreamer` throws `InvalidOperationException` if `IMediator` is not already registered. The first parameter is the config section name (e.g., `"SharpStreamerSettings"`), and the assemblies are scanned for `[ConsumeEvent]`-attributed types that implement `IRequest`.

### Key Interfaces and Types

**`IStreamerBus`** (`Bus/IStreamerBus.cs`) — the main publishing interface, registered as scoped:
- `PublishAsync<T>(T message, string eventKey, params KeyValuePair<string, string>[] headers)` — stores event in outbox within the caller's DB transaction. The `eventKey` controls ordering.
- `PublishDelayedAsync<T>(T message, TimeSpan delay, params KeyValuePair<string, string>[] headers)` — schedules event with a delay. Uses a random GUID v7 as `eventKey` (no ordering guarantee). The delay is implemented by setting `SentAt = now + delay`; the publisher background service only picks events where `SentAt < now`.

**`Event<TId>`** (`Entities/Abstractions/Event.cs`) — abstract base with `Id`, `Topic`, `Content` (JSON string), `RetryCount`, `SentAt`, `Timestamp`, `Status`, `EventKey`.

**`PublishedEvent : Event<Guid>`** — outbox table entity, no extra properties.

**`ReceivedEvent : Event<Guid>`** — inbox table entity with additional: `Group`, `ErrorMessage`, `UpdateTimestamp`, `Partition`, `NextExecutionTimestamp`.

**`EventStatus` enum:** `None = 0`, `InProgress = 1`, `Succeeded = 2`, `Failed = 3`.

### Attributes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SharpStreamer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
