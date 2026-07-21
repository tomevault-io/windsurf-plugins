---
trigger: always_on
description: Generates, for each actor:
---

# CloudActors – Implementation Reference

This document captures design decisions, project structure, and implementation details for contributors and AI agents working on the CloudActors codebase.

## Project Overview

CloudActors is an opinionated, simplified actor library for .NET built on Microsoft Orleans. It replaces Orleans' RPC-style grain API with a uniform message-passing style. All actor interactions go through a single `IActorBus` with just two operations: **Execute** and **Query**.

The library makes Orleans grains completely transparent to the developer: actors are plain C# classes annotated with `[Actor]`, and all Orleans plumbing (grain definition, serialization, activation, state persistence) is handled by Roslyn source generators.

`[Actor]` is the default snapshot-backed model. Event-sourced actors can additionally opt into generated `JournaledGrain` wrappers via `[Journaled]`, while keeping the actor itself as a POCO.

Orleans version is centralized in `src/Directory.props` as `$(OrleansVersion)`. Currently targeting **Orleans 10.1.0** and **net10.0**.

---

## Repository Layout

```
src/
├── CloudActors.Abstractions/           # Core public abstractions (IActorBus, IActorCommand, etc.)
├── CloudActors.Abstractions.CodeAnalysis/  # Source generators + analyzers that run in actor domain projects
├── CloudActors.Abstractions.CodeFix/   # Roslyn code fixes for diagnostics
├── CloudActors.Abstractions.Package/   # NuGet packaging for Abstractions
├── CloudActors.Package/                # NuGet meta-package (Devlooped.CloudActors)
├── CloudActors.Hosting/                # Runtime: OrleansActorBus, state factory, DI extensions
├── CloudActors.Hosting.CodeAnalysis/   # Source generators that run in Orleans silo projects
├── CloudActors.Streamstone/            # Optional Streamstone/Azure Table Storage grain storage provider
├── CloudActors.Streamstone.CodeAnalysis/ # Streamstone-specific journaled storage light-up generators
├── TestApp/                            # End-to-end ASP.NET + Orleans host sample
├── TestDomain/                         # Sample actor domain library (used by tests)
├── Tests/                              # Integration tests (Orleans in-process)
├── Tests.CodeAnalysis/                 # Incremental source-generator tests
└── Tests.CodeFix/                      # Roslyn code-fix tests
```

---

## Core Abstractions (`CloudActors.Abstractions`)

### IActorBus

The single entry point for all actor interactions, injected via DI:

```csharp
public interface IActorBus
{
    Task ExecuteAsync(string id, IActorCommand command);
    Task<TResult> ExecuteAsync<TResult>(string id, IActorCommand<TResult> command);
    Task<TResult> QueryAsync<TResult>(string id, IActorQuery<TResult> query);
}
```

- **ExecuteAsync** — state-changing operation (maps to a regular Orleans grain call)
- **QueryAsync** — read-only operation (maps to `[ReadOnly]` Orleans grain method)
- `id` is a string like `"account/42"`; typed-ID overloads are generated per actor

The three public methods are **default interface methods** that delegate to hidden `[OverloadResolutionPriority(1)]` overloads with `[CallerMemberName]`, `[CallerFilePath]`, and `[CallerLineNumber]` parameters used for automatic OpenTelemetry telemetry capture (see `Telemetry.cs`).

### Message interfaces

```csharp
IActorMessage            // base marker interface for all messages
IActorCommand            // void command (state-changing, no return value) — extends IActorMessage
IActorCommand<TResult>   // state-changing command with return value — extends IActorCommand
IActorQuery<TResult>     // read-only query — extends IActorMessage
```

Actor messages should be `partial record` types (required for source generators to add `[GenerateSerializer]`).

### [Actor] attribute

`ActorAttribute` flags a class as an actor. Optional parameters:
- `stateName` — overrides the default state name (defaults to the class name)
- `storageProvider` — Orleans storage provider name (defaults to the default provider)

### [Journaled] attribute

`JournaledAttribute` is an opt-in for event-sourced actors that should generate a `JournaledGrain<TView, TEvent>` wrapper instead of the default snapshot-backed grain.

- Requires the actor to also be `[Actor]` and list `IEventSourced`
- `ProviderName` overrides the Orleans log consistency/storage provider name when needed
- `BackgroundSave` switches journaled actors from the default auto-`await ConfirmEvents()` behavior to manual confirmation mode (`[Journaled(backgroundSave: true)]`)

### IEventSourced

Interface implemented by actors that use event sourcing. The implementation is **code-generated**; actors only need to list it in the base type list without implementing it. The generator provides:
- `Raise<T>(event)` — records an event and calls the matching `partial void Apply(TEvent)` method
- `Events`, `AcceptEvents()`, `LoadEvents()` — managed by the storage provider
- `ConfirmEvents()` — a generated protected helper available on every event-sourced actor. It is a safe no-op unless the hosting runtime/provider wires a confirmation callback.

### IActorIdFactory


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devlooped/CloudActors](https://github.com/devlooped/CloudActors) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
