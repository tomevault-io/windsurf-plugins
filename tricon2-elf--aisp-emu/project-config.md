---
trigger: always_on
description: | ------ | --------- |
---

# AGENTS.md

## Quick reference

| Task | Command |
| ------ | --------- |
| Restore + build | `dotnet restore aisp.sln && dotnet build aisp.sln` |
| Run tests (all) | `dotnet test aisp.sln` |
| Run single test project | `dotnet test aisp.Common.Tests` |
| Collect coverage | `./scripts/run-coverage.sh` |
| Coverage HTML report | `./scripts/reportgenerator.sh` |
| Run the server | `dotnet run --project aisp.Server` |
| Format staged files | `git config core.hooksPath .githooks` (once), then commit |
| Check format (CI style) | `dotnet tool restore && dotnet tool run csharpier check .` |
| Format everything | `dotnet tool restore && dotnet tool run csharpier format .` |
| EF migration | `./scripts/generate-migration.sh <Name>` |
| Docker build | `docker compose up -d` |

## Toolchain

- **.NET 10 SDK** required (all projects target `net10.0` — this is a preview SDK).
- **CSharpier** for formatting: `dotnet tool restore` installs it locally. Config: `printWidth: 400` (intentionally wide).
- **EF Core 10.0**: migrations in `aisp.Common/DAL/Migrations/`, tool `dotnet-ef` managed in `dotnet-tools.json`.
- **Coverage**: Coverlet collector on test projects; `./scripts/run-coverage.sh` then `./scripts/reportgenerator.sh` (ReportGenerator via `dotnet-tools.json`).
- **Tests use xunit.v3** (not v2). Test projects have `OutputType Exe` and global `using Xunit`. Moq for mocking, SQLite in-memory for DB integration tests. Use `TestContext.Current.CancellationToken` for test cancellation.

## Project dependency graph (strict — no cycles)

```text
aisp.Network  (no project deps — wire format + transport only)
       ↑
aisp.Common   (game logic, EF Core DAL, packet handlers)
       ↑
aisp.Server   (executable; ASP.NET Core host + BackgroundServices)
```

Each layer must not reference anything above it. `aisp.Network` has zero game logic or DB entities.

## Architecture

### Single process, three domain servers

All three game servers run as `BackgroundService` instances inside one process:

| Server | Port | ServerType enum |
| -------- | ------ | ----------------- |
| Auth | 50050 | `ServerType.Auth` (1) |
| Msg | 50052 | `ServerType.Msg` (3) |
| Area | 50054 | `ServerType.Area` (2) |

Each derives from `GameServerBase<T>` (`GameServerBase.cs:50`) which owns a TCP `VceListener`, a `Channel<Packet>` dispatch loop, and a 60 Hz tick timer.

### Packet handler pattern

Packet handlers implement `IPacketHandler` (`PacketHandlerBase.cs:21`) with three properties:

- `RequestType` (`PacketType` enum)
- `ResponseType` (`PacketType` enum)
- `ServerType` (`ServerType` enum)

They are auto-discovered by Scrutor at startup (no manual registration needed). Place new handlers in `aisp.Common/Handlers/<Domain>/`.

The generic base class `PacketHandlerBase<TRequest, TResponse>` deserializes the request from bytes and serializes the response automatically. If a handler returns `null`, no response is sent.

### Packet types

`PacketType.cs` in `aisp.Network/` — a master enum (~600 entries) annotated with `[PacketMetadata]` attributes specifying domain, ID, and name.

### Adding a new packet/feature

1. Packet DTOs in `aisp.Network/Packets/<Domain>/` (implement `IIncomingPacket<T>` / `IOutgoingPacket`)
2. Entry in `aisp.Network/PacketType.cs`
3. Handler class in `aisp.Common/Handlers/<Domain>/` (auto-discovered)
4. Persistence in `aisp.Common/DAL/` if needed, then generate migration

## Database

- **Default**: SQLite at `db/main.db` (relative to the process working directory). Override via `Server:DbOptions` in config or `Server__DbOptions__ConnectionString` (Docker: `/data/main.db` with compose volume `aisp-data` → `/data`).
- **Also supports**: SQL Server (packages are referenced).
- **Migrations auto-applied** at startup via `db.Database.MigrateAsync()` in `Program.cs:193`.
- **Seeding**: `Program.cs` calls `Seed*IfEmptyAsync` helpers on repositories for maps, map links, worlds, channels. Items are seeded once at startup in `Program.cs` (after migrations) via `ItemRepository.SeedItemsIfEmptyAsync` from `seedData/baseItems.json` (under `aisp.Common/`, copied to output). Localised catalog names use inline `LocalisedString` objects (`ja`/`en`/`zh-Hans`/`zh-Hant`); `LocalisationCatalogSeeder` upserts missing `LocalisedTexts` rows. Runtime code reads items from the database only; display strings go through `ITextLocaliser`.
- **Integration tests**: Use `TestDb.CreateInMemoryMainContext()` (`tests/aisp.Common.Tests/Support/TestDb.cs`) for a disposable SQLite in-memory context.
- **Migration command** (from repo root):

  ```bash
  dotnet ef migrations add <Name> --project aisp.Common/aisp.Common.csproj --startup-project aisp.Server/aisp.Server.csproj --context MainContext --output-dir DAL/Migrations
  ```

  Or use `./scripts/generate-migration.sh <Name>`.

## Configuration

- `appsettings.json` in `aisp.Server/` — copied to output on build (`PreserveNewest`).
- `IP_OVERRIDE` env var (or `Server__IPOverride` config key) replaces `localhost` in server addresses (required for Docker).
- `ApiSettings.ApiKey` — API endpoints under `/api/` require `X-Api-Key` header matching this value.
- Per-server enable/disable and port via `Server:AuthServer`, `Server:MsgServer`, `Server:AreaServer` config sections.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Tricon2-Elf/aisp-emu](https://github.com/Tricon2-Elf/aisp-emu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
