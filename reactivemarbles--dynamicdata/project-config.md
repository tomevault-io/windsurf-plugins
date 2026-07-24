---
trigger: always_on
description: DynamicData is a reactive collections library for .NET, built on top of [Reactive Extensions (Rx)](https://github.com/dotnet/reactive). It provides `SourceCache<TObject, TKey>` and `SourceList<TObject>` — observable data collections that emit **changesets** when modified. These changesets flow through operator pipelines (Sort, Filter, Transform, Group, Join, etc.) that maintain live, incrementally-updated views of the data.
---

# DynamicData — AI Instructions

## What is DynamicData?

DynamicData is a reactive collections library for .NET, built on top of [Reactive Extensions (Rx)](https://github.com/dotnet/reactive). It provides `SourceCache<TObject, TKey>` and `SourceList<TObject>` — observable data collections that emit **changesets** when modified. These changesets flow through operator pipelines (Sort, Filter, Transform, Group, Join, etc.) that maintain live, incrementally-updated views of the data.

DynamicData is used in production by thousands of applications. It is the reactive data layer for [ReactiveUI](https://reactiveui.net/), making it foundational infrastructure for the .NET reactive ecosystem.

## Cache vs List — Two Collection Types

DynamicData provides two parallel collection types. **Choose the right one — they are not interchangeable.**

| | **Cache** (`SourceCache<T, TKey>`) | **List** (`SourceList<T>`) |
|---|---|---|
| **Identity** | Items identified by unique key | Items identified by index position |
| **Duplicates** | Not allowed (key must be unique) | Allowed (same item at multiple positions) |
| **Ordering** | Unordered by default (Sort adds ordering) | Inherently ordered (like `List<T>`) |
| **Best for** | Entities with IDs, lookup by key | Ordered sequences, duplicates OK |
| **Change types** | Add, Update, Remove, Refresh, Moved | Add, AddRange, Replace, Remove, RemoveRange, Moved, Refresh, Clear |
| **Changeset** | `IChangeSet<TObject, TKey>` | `IChangeSet<T>` |

**Rule of thumb:** If your items have a natural unique key (ID, name, etc.), use **Cache**. If order matters and/or duplicates are possible, use **List**. Cache is used far more often in practice.

See `.github/instructions/dynamicdata-cache.instructions.md` for the complete cache operator reference.

See `.github/instructions/dynamicdata-list.instructions.md` for the complete list operator reference.

## Why Performance Matters

Every item flowing through a DynamicData pipeline passes through multiple operators. Each operator processes changesets — not individual items — so a single cache edit with 1000 items creates a changeset that flows through every operator in the chain. At library scale:

- **Per-item overhead compounds**: 1 allocation × 10 operators × 1000 items × 100 pipelines = 1M allocations per batch
- **Lock contention is the bottleneck**: operators serialize access to shared state. Minimizing lock hold time is a core design goal.
- **Prefer value types and stack allocation**: use structs, `ref struct`, `Span<T>`, and avoid closures in hot paths where possible

When optimizing, measure allocation rates and lock contention, not just wall-clock time.

## Why Rx Contract Compliance is Critical

DynamicData operators compose — the output of one is the input of the next. If any operator violates the Rx contract (e.g., concurrent `OnNext` calls, calls after `OnCompleted`), every downstream operator can corrupt its internal state. This is not a crash — it's silent data corruption that manifests as wrong results, missing items, or phantom entries. In a reactive UI, this means the user sees stale or incorrect data with no error message.

See `.github/instructions/rx.instructions.md` for comprehensive Rx contract rules, scheduler usage, disposable patterns, and a complete standard Rx operator reference.

## Breaking Changes

DynamicData follows [Semantic Versioning (SemVer)](https://semver.org/). Breaking changes **are possible** in major version bumps, but they are never done lightly. This library has thousands of downstream consumers — every breaking change has a blast radius.

**Rules:**
- Breaking changes require a major version bump. **You MUST explicitly call out any potentially breaking change to the user** before making it — even if you think it's minor. Let the maintainers decide.
- Prefer non-breaking alternatives first: new overloads, new methods, optional parameters with safe defaults.
- When a breaking change is justified, mark the old API with `[Obsolete("Use XYZ instead. This will be removed in vN+1.")]` in the current version and remove it in the next major.
- Behavioral changes (different ordering, different filtering semantics, different error propagation) are breaking even if the signature is unchanged. Call these out.
- Internal types (`internal` visibility) can change freely — they are not part of the public contract.

**What counts as breaking:**
- Changing the signature of a public extension method (parameters, return type, generic constraints)
- Changing observable behavior (emission order, filtering semantics, error/completion propagation)
- Removing or renaming public types, methods, or properties
- Adding required parameters to existing methods
- Changing the default behavior of an existing overload

## Maintaining These Instructions

These instruction files are living documentation. **They must be kept in sync with the code.**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reactivemarbles/DynamicData](https://github.com/reactivemarbles/DynamicData) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
