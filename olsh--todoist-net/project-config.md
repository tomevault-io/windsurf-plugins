---
trigger: always_on
description: Guidance for coding agents working in this repository.
---

# AGENTS.md

Guidance for coding agents working in this repository.

## Project Overview

Todoist.Net is a .NET client library for the **Todoist unified API v1** (`https://api.todoist.com/api/v1/`, set in `TodoistRestClient`). It wraps both the batch `sync` endpoint and the plain REST v1 endpoints behind one strongly-typed client.

The repo is mid-migration from the old Sync API v9 (`11.0.0-beta.1`; see the "API v1 Migration - Phase 1/2" commits). Public naming was moved to v1 vocabulary (`Item` -> `Task`, `Note` -> `Comment`) while the sync wire protocol kept the legacy names — see [Naming: C# vs. wire](#naming-c-vs-wire) before touching models or commands.

Library targets `netstandard2.0` and `net462`; the test project targets `net8.0`.

## Build and Test Commands

NUKE is the build system. Targets are invoked in **kebab-case** on the CLI:

```bash
.\build.cmd                                    # default chain: compile, unit-test, nuget-pack
.\build.cmd unit-test                          # trait=unit only (no API token needed)
.\build.cmd test                               # everything except trait!=mfa-required (hits the live API)
.\build.cmd nuget-pack --configuration Release # package into artifacts/
```

`build.cmd` (Windows) and `build.sh` (Linux/macOS) both delegate to `build/Build.cs`. Configuration defaults to `Debug` locally, `Release` on CI.

Direct dotnet CLI:

```bash
dotnet build Todoist.Net.slnx
dotnet test src/Todoist.Net.Tests/Todoist.Net.Tests.csproj --filter "trait=unit"
dotnet test src/Todoist.Net.Tests/Todoist.Net.Tests.csproj --filter "FullyQualifiedName~TransactionTests"
```

`dotnet test` only builds the `netstandard2.0` leg. Run `dotnet build Todoist.Net.slnx` to compile `net462` too — a change can pass tests and still break that target framework.

Integration tests read the token from the **`todoist:token`** environment variable (`Settings/SettingsProvider.cs`) and mutate a real Todoist account.

## Architecture

### Two transports behind one client

`TodoistClient` implements the public `ITodoistClient` plus the internal `IAdvancedTodoistClient`. All HTTP goes through the latter, which services consume:

- **Batch/sync** — `ExecuteCommandsAsync` POSTs a `commands` array to `sync`, and `GetResourcesAsync` POSTs `sync_token` + `resource_types` to the same endpoint. Server errors arrive per-command in `sync_status` and are re-thrown as an `AggregateException` of `TodoistException`.
- **REST v1** — `GetAsync<T>` / `PostJsonAsync<T>` / `PutJsonAsync<T>` / `PostFormAsync<T>` and the raw `GetRawAsync` / `PostRawAsync` / `DeleteRawAsync` variants hit named resources (`tasks/{id}`, `projects/archived`, `uploads`, `templates/file`, …).

`IAdvancedTodoistClient` is `internal` and implemented explicitly, so the transport surface never leaks to consumers. Widening it is a deliberate decision, not a convenience.

### Command pattern and transactions

`Command` = `{ type, args, temp_id, uuid }`, where `type` is a `CommandType` and `args` any `ICommandArgument`.

`CommandServiceBase` gives every command service two modes, selected by which constructor ran:

- **Direct** — built with an `IAdvancedTodoistClient`; `ExecuteCommandAsync` fires one HTTP request immediately.
- **Queued** — built with an `ICollection<Command>`; `ExecuteCommandAsync` just appends to the transaction's list.

`Transaction` owns the `LinkedList<Command>` and constructs a queued instance of every command service. `CommitAsync` sends the batch and clears the queue (in a `finally`, so a failed commit does not leave commands to be re-sent).

```csharp
var transaction = client.CreateTransaction();
var projectId = await transaction.Project.AddAsync(new Project("New project"));
var taskId = await transaction.Tasks.AddAsync(new AddTask("New task", projectId));
await transaction.Comments.AddToTaskAsync(new Comment("Description"), taskId);
await transaction.CommitAsync();
```

### Service layer

Per resource there are two types, and **`{Resource}Service` inherits `{Resource}CommandService`** — that inheritance is why `client.Tasks` exposes reads and writes together while `transaction.Tasks` exposes only the queueable writes:

- `{Resource}CommandService : CommandServiceBase, I{Resource}CommandService` — sync commands (`AddAsync`, `UpdateAsync`, `DeleteAsync`, …); usable in both modes.
- `{Resource}Service : {Resource}CommandService, I{Resource}Service` — adds the read/REST calls; requires a client, so it is direct-mode only.

`TodoistClient`'s constructor instantiates the `*Service` variants; `Transaction`'s constructor instantiates the `*CommandService` variants. Adding a resource means wiring both.

### ID handling: ComplexId and temp IDs

`ComplexId` is a struct holding either a `PersistentId` (server string) or a `TempId` (`Guid`). `CreateAddCommand` assigns a fresh temp ID to a new entity and sends it as the command's `temp_id`, letting later commands in the same batch reference an entity that does not exist yet. After a commit, `UpdateTempIds` walks the sent commands and rewrites `BaseEntity.Id` from the response's `temp_id_mapping`; arguments that reference *other* entities implement `IWithRelationsArgument` to patch their own references.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [olsh/todoist-net](https://github.com/olsh/todoist-net) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
