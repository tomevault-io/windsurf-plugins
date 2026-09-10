---
trigger: always_on
description: FIO is a type-safe, purely functional effect system for F#. IO monad + fibers (green threads) for concurrent/async apps.
---

# Copilot Instructions

FIO is a type-safe, purely functional effect system for F#. IO monad + fibers (green threads) for concurrent/async apps.

**Target:** .NET 10, F# 10, `.slnx` solution format

## Build & Test

```bash
dotnet build                                    # Build all
dotnet test                                     # Run all tests
dotnet test tests/FIO.Tests/                    # Core tests only
dotnet test tests/FIO.Sockets.Tests/            # Sockets tests only
dotnet test tests/FIO.WebSockets.Tests/         # WebSockets tests only
dotnet test --filter "Name~TestName"            # Run specific test
dotnet test --filter "Name~PropertyTests"       # Run test file/group

# Examples (five projects: DSL, App, Http, Sockets, WebSockets)
dotnet run --project examples/FIO.Examples.DSL

# Benchmarks (Release mode, BenchmarkDotNet)
dotnet run -c Release --project benchmarks/FIO.Benchmarks -- --filter "*"           # all benchmarks
dotnet run -c Release --project benchmarks/FIO.Benchmarks -- --list flat            # list benchmarks
# Configure via env vars: FIO_BENCH_RUNTIMES, FIO_BENCH_ITERATIONS, FIO_BENCH_<NAME>_ROUNDS/ACTORS
```

## Architecture

### FIO Type

`FIO<'A, 'E>` is a discriminated union representing lazy effects. DU cases are `internal` — external code uses factory functions and instance methods.

Key DU cases: `Success`/`Failure` (terminal), `Interrupt` (self-interrupt), `Action` (sync side effects), `WriteChan`/`ReadChan` (channels), `ForkEffect` (forking), `JoinFiber` (waiting), `JoinFirst` (first of several fibers to settle), `JoinAllFailFast` (all fibers, settling early on first failure), `AwaitTask` (.NET Task interop), `ChainSuccess`/`ChainError`/`ChainBoth` (bind), `OnFinalize` (finalizer infrastructure), `FiberCancellationToken` (current fiber token), `Suspend` (deferred construction).

### Runtime Hierarchy

```
FIORuntime (abstract)
├── DirectRuntime              — .NET Tasks, waits for blocked fibers
└── FIOWorkerRuntime (abstract, adds WorkerConfig: EvaluationWorkers/EvaluationSteps/BlockingWorkers)
    ├── PollingRuntime         — Custom fibers, linear-time blocked handling (polling)
    ├── SignalingRuntime       — Custom fibers, event-driven blocked handling (dedicated BlockingWorker + signal queue; constant-time). Comparison/legacy runtime
    └── WorkStealingRuntime    — Custom fibers, work-stealing scheduler (per-worker queues + work-stealing). The default
```

`DefaultRuntime = WorkStealingRuntime` (recommended). Worker config fields: **EvaluationWorkers** (evaluation worker count), **EvaluationSteps** (eval steps per work item before rescheduling), **BlockingWorkers** (used by `PollingRuntime`; **ignored by `WorkStealingRuntime`**). The `EWC`/`EWS`/`BWC` acronyms remain as the `ConfigString` display labels and benchmark spec shorthand.

### Key Files (compile order matters)

Core DSL (`src/FIO/DSL/`):
- `Core.fs` — `FIO<'A,'E>` DU, `Fiber`, `Channel`, `FiberContext`, `WorkItem`, `ContStack`. Hosts primitive instance members: `FlatMap`, `CatchAll`, `Ensuring`, `Fork` plus the transformation cluster (`Map`/`MapError`/`MapBoth`/`Result`/`Option`/`Choice`) derived from `Success`/`Failure` + the four primitives.
- `Factories.fs` / `Extensions.fs` / `Operators.fs` / `CE.fs` — public surface built on Core.

Console I/O (`src/FIO/Console.fs`):
- `FIO.Console.Console` (`[<RequireQualifiedAccess>]`): `print`, `printLine`, `readLine`, `write`, `writeLine`, `clear`, each taking `onError: exn -> 'E`.

Runtime (`src/FIO/Runtime/`):
- `Runtime.fs` — `FIORuntime` base, `WorkerConfig`, `ContStackPool`, `WorkItemPool`
- `WorkerInfrastructure.fs` — `FIOWorkerRuntime`, `WorkerLifecycle`
- `InterpreterCore.fs` — shared interpreter (`InterpreterState`, `processOutcome`/`processResult`/`handleSharedCase`, `Outcome` DU, `RuntimeCase` DU for runtime-specific dispatch, park helpers for the `JoinFirst`/`JoinAllFailFast` primitives)
- `DirectRuntime.fs` / `PollingRuntime.fs` / `SignalingRuntime.fs` / `WorkStealingRuntime.fs` / `DefaultRuntime.fs` (alias for `WorkStealingRuntime`)

Framework (`src/FIO/App.fs`): `FIOApp<'A,'E>` with 7-member surface (`effect`, `runtime`, `onOutcome`, `onOutcomeTimeout`, `onShutdown`, `onShutdownTimeout`, `mapExitCode`) over `AppResult` (`AppSucceeded`/`AppFailed`/`AppInterrupted`/`AppFatalError`).

### Concurrency Primitives

- **Fiber<'A,'E>** — green thread via `.Fork()` / `.Join()`
- **Channel<'A>** — typed message passing between fibers
- **InterruptionCause** — `ParentInterrupted` | `ExplicitInterrupt` | `InvalidArgument` | `ResourceExhaustion`

### Packages

- **FIO** — Core (effect system, fibers, channels, runtimes, App framework, Console)
- **FIO.Sockets** — TCP sockets (error type: `SocketError`)
- **FIO.WebSockets** — WebSockets (error type: `WsError`)
- **FIO.Http** — HTTP server, Kestrel-based (error type: `HttpError`)

## Conventions

### API Naming

- Factory functions use **lowercase** F#-idiomatic style: `FIO.succeed`, `FIO.fail`, `FIO.attempt`, `FIO.sleep`, `FIO.collectAllPar`, `FIO.acquireReleaseWith`
- Instance methods use **PascalCase**: `effect.Map(f)`, `effect.FlatMap(f)`, `effect.Fork()`, `effect.CatchAll(f)`, `effect.Ensuring(fin)`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fs-fio/FIO](https://github.com/fs-fio/FIO) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
