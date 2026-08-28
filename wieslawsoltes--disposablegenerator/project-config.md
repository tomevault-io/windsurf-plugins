---
trigger: always_on
description: This file is the repository-wide implementation contract. It applies to every source, test, sample, package, documentation, and workflow change. Treat **must**, **must not**, and **required** as mandatory.
---

# DisposableGenerator agent contract

This file is the repository-wide implementation contract. It applies to every source, test, sample, package, documentation, and workflow change. Treat **must**, **must not**, and **required** as mandatory.

## Project intent

DisposableGenerator is a compile-time-only incremental C# source generator for explicit `IDisposable` and `IAsyncDisposable` ownership. It must remove mechanical disposal code without hiding ownership decisions or weakening the conventional .NET disposal contract.

The package must remain analyzer-only: consumers receive generated source and injected attributes, with no DisposableGenerator runtime assembly reference or runtime dependency.

## Toolchain and repository layout

- Use the SDK selected by `global.json` and central versions from `Directory.Packages.props`.
- Keep the generator in `src/DisposableGenerator`, unit/generator tests in `tests/DisposableGenerator.Tests`, package-consumer tests in `tests/PackageIntegration`, samples in `samples`, public documentation in `README.md` and `docs`, and automation in `eng` and `.github/workflows`.
- Keep the generator target at `netstandard2.0` unless a deliberate compatibility change is approved and documented.
- Preserve nullable analysis, deterministic builds, warnings-as-errors, and the repository formatting rules.
- Do not add a runtime helper library. Generated consumers must depend only on their target framework.

## Public ownership contract

- Generation is opt-in through `[GenerateDisposable]` on a non-static, non-record partial class. Every containing type must also be partial. File-local owners or containers are unsupported.
- Ownership is always explicit. A disposable member is owned only when marked `[DisposeMember]`; `[BorrowedMember]` records that another component owns it. Never infer ownership from a disposable type, constructor assignment, dependency injection, naming, accessibility, or mutability.
- Resolve `DISP006` by making the ownership decision visible. Use `[DisposeMember]` for resources this instance must release and `[BorrowedMember]` for DI-, framework-, parent-, cache-, or otherwise externally owned resources. Do not add `#pragma warning disable DISP006` as the normal fix. Suppression is permitted only when neither annotation can express the scenario, and the code must document why.
- An owned target must be a readable instance field or property whose type implements `IDisposable` or `IAsyncDisposable`. A target must never be both owned and borrowed.
- Preserve `[DisposeMember(Order = value)]`: lower values dispose first; equal values use configured declaration order.
- Warn for mutable owned members because replacing their value can leak the former resource. Do not silently snapshot or intercept assignments.

## Generated API and lifetime state

- A synchronous root implements `IDisposable`, exposes non-virtual public `Dispose()`, and supplies the conventional overridable `Dispose(bool)` core for generated inheritance.
- An asynchronous root implements `IAsyncDisposable`, exposes public `DisposeAsync()`, and supplies `DisposeAsyncCore()` for generated inheritance. A conjunctive root implements both contracts; an async-only root must not advertise `IDisposable`.
- Generated derived levels must override and chain the generated base core. Only the root owns public entry points, lifetime state, the synchronization gate, and the dynamic-registration collection.
- `Dispose()` and `DisposeAsync()` must share one atomic lifetime transition. Repeated and competing calls, including concurrent sync/async calls, must execute cleanup at most once.
- Mark the instance disposed before invoking user cleanup. Registration must not race into a collection being drained.
- Every public deterministic disposal entry point must call `GC.SuppressFinalize(this)` after its generated cleanup path.
- Generated member names, accessibility, and signatures are compatibility surface. Detect collisions and report a diagnostic instead of overwriting, hiding, or ambiguously binding handwritten members.

## Disposal modes

- `GenerateSynchronousDispose` defaults to `true`; async, unmanaged cleanup, and finalization default to `false`.
- At least one disposal interface must be enabled. Finalization requires synchronous generation and implies unmanaged cleanup.
- Every generated level in an inheritance chain must use the same synchronous/asynchronous interface mode.
- Async cleanup prefers `IAsyncDisposable.DisposeAsync()` for dual-capability resources and falls back to `IDisposable.Dispose()` for sync-only resources.
- Sync cleanup must never block on async-only cleanup. Diagnose fixed async-only members on a conjunctive owner with `DISP022`; require async generation with `DISP020` when no async path exists.
- `RegisterAsyncDisposable` returns `ValueTask<T>` because late registration may require immediate asynchronous disposal. Call sites must await it.

## Ordering, hooks, and exceptions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wieslawsoltes/DisposableGenerator](https://github.com/wieslawsoltes/DisposableGenerator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
