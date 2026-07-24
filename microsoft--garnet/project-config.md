---
trigger: always_on
description: Garnet is a high-performance remote cache-store from Microsoft Research implementing the Redis RESP wire protocol in C#/.NET. It uses Tsavorite as its storage engine. Full developer docs: https://microsoft.github.io/garnet/docs/dev/onboarding
---

# Garnet - Copilot Instructions

Garnet is a high-performance remote cache-store from Microsoft Research implementing the Redis RESP wire protocol in C#/.NET. It uses Tsavorite as its storage engine. Full developer docs: https://microsoft.github.io/garnet/docs/dev/onboarding

> **Note**: Some website docs may reference the older two-store architecture (separate main store and object store). This branch uses a **unified single-store** design — see the Architecture section below for the current model.

## Build, Test, and Lint

```bash
# Build the entire solution
dotnet build

# Run all Garnet tests
dotnet test test/Garnet.test -f net10.0 -c Debug -l "console;verbosity=detailed"

# Run all cluster tests
dotnet test test/Garnet.test.cluster -f net10.0 -c Debug -l "console;verbosity=detailed"

# Run a single test by fully qualified name
dotnet test test/Garnet.test -f net10.0 -c Debug --filter "FullyQualifiedName~RespTests.PingTest"

# Run all tests in a single test class
dotnet test test/Garnet.test -f net10.0 -c Debug --filter "FullyQualifiedName~RespTests"

# Build and test Tsavorite independently (has its own solution)
dotnet build libs/storage/Tsavorite/cs/test/Tsavorite.test.csproj
dotnet test libs/storage/Tsavorite/cs/test/Tsavorite.test.csproj -f net10.0 -c Debug -l "console;verbosity=detailed"

# Check formatting (CI enforces this)
dotnet format Garnet.slnx --verify-no-changes
dotnet format libs/storage/Tsavorite/cs/Tsavorite.slnx --verify-no-changes

# Run the server locally (from repo root)
cd main/GarnetServer && dotnet run -c Debug -f net10.0 -- --logger-level Trace -m 4g -i 64m
```

Target frameworks are `net8.0` and `net10.0`. CI runs tests on both, in Debug and Release, on Ubuntu and Windows.

## Architecture

### Unified Single-Store Design

Garnet uses a **single Tsavorite key-value store** instance (`TsavoriteKV<StoreFunctions, StoreAllocator>`) that holds both raw strings and complex objects. The store is accessed through three different **context types**, each with its own input/output types and session functions:

| Context | Input/Output Types | Session Functions | Used For |
|---------|-------------------|-------------------|----------|
| **String context** | `StringInput` / `StringOutput` | `MainSessionFunctions` | Raw string commands (GET, SET, APPEND, INCR, etc.) |
| **Object context** | `ObjectInput` / `ObjectOutput` | `ObjectSessionFunctions` | Collection commands (HSET, LPUSH, ZADD, SADD, etc.) |
| **Unified context** | `UnifiedInput` / `UnifiedOutput` | `UnifiedSessionFunctions` | Type-agnostic commands (EXISTS, DELETE, TYPE, TTL, EXPIRE, RENAME, etc.) |

All three contexts operate on the **same underlying store**. At the storage level, each record's `RecordInfo` has a `ValueIsObject` bit that indicates whether the value is a raw string (inline bytes) or a heap object reference, enabling the unified store to differentiate between the two value types. The `GarnetApi` struct is generic over all three context types:

```csharp
public partial struct GarnetApi<TStringContext, TObjectContext, TUnifiedContext>
```

Two concrete instantiations are used: `BasicGarnetApi` (normal operations) and `TransactionalGarnetApi` (within transactions). Type aliases for all context variants are defined in `libs/GlobalUsings.cs`.

The single store is held by `GarnetDatabase` (`libs/server/GarnetDatabase.cs`) and managed by `StoreWrapper` (`libs/server/StoreWrapper.cs`). Each record carries a `ValueIsObject` bit in its `RecordInfo` header to distinguish raw string values from serialized object values.

#### Storage layer organization

Each context type has parallel directory structures:

- **Functions** (Tsavorite callbacks for RMW, Read, Upsert, Delete):
  - `libs/server/Storage/Functions/MainStore/` — string operations
  - `libs/server/Storage/Functions/ObjectStore/` — collection operations
  - `libs/server/Storage/Functions/UnifiedStore/` — type-agnostic operations
- **Session ops** (StorageSession methods wrapping Tsavorite API):
  - `libs/server/Storage/Session/MainStore/` — string ops (MainStoreOps.cs, BitmapOps.cs, HyperLogLogOps.cs)
  - `libs/server/Storage/Session/ObjectStore/` — collection ops ([ObjectName]Ops.cs)
  - `libs/server/Storage/Session/UnifiedStore/` — unified ops (UnifiedStoreOps.cs)
- **Object implementations**: `libs/server/Objects/[ObjectName]/` — per-type logic (Hash, List, Set, SortedSet, SortedSetGeo)

### Key Layers

- **Network/Session** (`libs/common/Networking/`, `libs/server/Sessions/`) — Shared-memory network design where TLS and storage ops run on IO completion threads. `GarnetServerTcp` accepts connections, creates `ServerTcpNetworkHandler` per client. `GarnetProvider` creates `RespServerSession` instances to handle RESP messages.
- **RESP Command Processing** (`libs/server/Resp/`) — Commands are defined as `RespCommand` enum values and dispatched via switch expressions in `ProcessBasicCommands`/`ProcessArrayCommands`. The `RespServerSession` class is split across multiple partial `.cs` files organized by command category.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/garnet](https://github.com/microsoft/garnet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
