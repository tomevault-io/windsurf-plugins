---
trigger: always_on
description: .NET distributed application framework providing in-process mediator and message bus capabilities for event-driven architectures. Part of the "critter stack" with Marten.
---

# Wolverine

.NET distributed application framework providing in-process mediator and message bus capabilities for event-driven architectures. Part of the "critter stack" with Marten.

## Tech Stack

- **Language**: C# 12+
- **Frameworks**: .NET 9.0, 10.0 (net8.0 dropped in 6.0; 5.x maintained on the `5.0` branch)
- **Build**: MSBuild + Nuke (scripted automation)
- **Core Dependencies**: JasperFx (runtime compilation), Microsoft.Extensions.*, System.Threading.Tasks.Dataflow
- **Docs**: Vitepress (in `/docs`)

## Project Structure

```
src/
├── Wolverine/              # Core framework - message bus, handlers, middleware
├── Http/                   # HTTP/REST support
│   ├── Wolverine.Http/     # ASP.NET Core integration
│   └── Wolverine.Http.Marten/
├── Persistence/            # Storage providers
│   ├── Wolverine.RDBMS/    # Base RDBMS implementation
│   ├── Wolverine.SqlServer/
│   ├── Wolverine.Postgresql/
│   ├── Wolverine.EntityFrameworkCore/
│   ├── Wolverine.Marten/   # Event sourcing with Marten
│   └── Wolverine.RavenDb/
├── Transports/             # Message brokers
│   ├── RabbitMQ/
│   ├── AWS/                # SQS, SNS
│   ├── Azure/              # Service Bus
│   ├── Kafka/
│   ├── Redis/
│   └── [NATS, Pulsar, GCP, MQTT, SignalR]
├── Extensions/             # Serialization & validation
│   ├── Wolverine.FluentValidation/
│   ├── Wolverine.MessagePack/
│   └── Wolverine.MemoryPack/
├── Samples/                # Example applications
└── Testing/                # Test suites
    ├── CoreTests/
    ├── Wolverine.ComplianceTests/
    └── SlowTests/
```

## Build & Test

```bash
# Build (uses Nuke)
./build.sh              # macOS/Linux
build.ps1               # Windows

# Start test infrastructure
docker compose up -d    # PostgreSQL, SQL Server, RabbitMQ, Kafka, etc.

# Run tests
dotnet test             # All tests
dotnet test src/Testing/CoreTests/

# Documentation
npm install && npm run docs
```

**Solutions** (new `.slnx` XML format):
- `wolverine.slnx` - Full solution
- `wolverine_slim.slnx` - Lightweight variant

> ⚠️ **CI builds `wolverine.slnx` (the full solution)** — pinned in
> `.nuke/parameters.json` (`"Solution": "wolverine.slnx"`) and consumed by
> the `Compile` target in `build/Build.cs`.
>
> `wolverine_slim.slnx` omits several extension projects — most notably
> `Wolverine.EntityFrameworkCore`, `Wolverine.Polecat`, `Wolverine.RavenDb`,
> `Wolverine.Http.Newtonsoft`, `Wolverine.FluentValidation`,
> `Wolverine.AzureServiceBus.Tests`, the `PolecatTests` / `MartenTests`
> suites, and `WolverineWebApi`. A clean slim build is **not sufficient
> evidence** that a tightening change to a dependency (e.g. a new AOT/trim
> annotation in JasperFx, a type lift from Marten/Polecat into JasperFx.Events)
> is clean across Wolverine. **Build `wolverine.slnx` before pushing**:
>
> ```bash
> dotnet build wolverine.slnx -c Release
> ```

## Key Entry Points

| Concept | Location |
|---------|----------|
| Options/Config | `src/Wolverine/WolverineOptions.cs:97` |
| Message Bus | `src/Wolverine/Runtime/MessageBus.cs` |
| Message Context | `src/Wolverine/Runtime/MessageContext.cs:34` |
| Handler Discovery | `src/Wolverine/Configuration/HandlerDiscovery.cs:17` |
| Endpoint Config | `src/Wolverine/Configuration/ListenerConfiguration.cs` |
| Sagas | `src/Wolverine/Saga.cs:8` |
| HTTP Endpoints | `src/Http/Wolverine.Http/` |

## Handler Conventions

Valid handler method names: `Handle`, `HandleAsync`, `Consume`, `ConsumeAsync` (`HandlerDiscovery.cs:17-22`)

Handlers are discovered by scanning assemblies. Use attributes like `[WolverineHandler]`, `[WolverineMessage]`, `[WolverineIgnore]` to control discovery.

## Naming conventions

Member casing is driven by **accessibility**, not by member kind:

- **`internal` and `public` members** (methods, properties, events, constants) use **PascalCase**.
- **`private` and `protected` members** (methods, properties) use **camelCase** — e.g. `buildSenderIfMissing()`,
  `writeOutgoingHeader()`, `tableExistsAsync()`. This includes `private static` helper methods.
- Private/protected **fields** keep the conventional leading underscore + camelCase (`_sender`, `_queueTable`).

When in doubt, match the surrounding file. Examples: `EnvelopeMapper` (`buildIncoming`/`buildOutgoing` private vs
`MapProperty`/`ReceivesMessage` public), `SqlServerQueue` (`buildSenderIfMissing` private vs `SendAsync` public).

## Performance conventions

### Use `ImHashMap` for hot-path dictionary lookups

For any dictionary lookup where performance matters — per-message work, per-Envelope work, per-handler dispatch — use `ImHashMap<TKey, TValue>` from `JasperFx.Core`. **Do not replace `ImHashMap` with `FrozenDictionary`** even when the data is post-bootstrap-immutable.

`ImHashMap` is a copy-on-write hash trie:
- Lookups are lock-free and don't allocate.
- Writes return a new map; callers swap via `Interlocked.CompareExchange` or a plain field assignment.
- The trie structure is friendlier to the JIT for our hot paths than `FrozenDictionary`'s hash-bucket layout in practice.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JasperFx/wolverine](https://github.com/JasperFx/wolverine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
