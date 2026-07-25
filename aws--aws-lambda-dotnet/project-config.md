---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`Amazon.Lambda.DurableExecution` is the .NET SDK (GA, 1.x) for resilient, long-running AWS Lambda
workflows that checkpoint progress after each step and resume after failures or waits. A workflow can run
for up to ~1 year (the WAIT cap is 31,622,400 seconds) and is only billed for active compute. The SDK is
client-side glue: the *durable execution service* (part of Lambda) owns the checkpoint store, fires timers,
and re-invokes the function; this library re-derives in-memory workflow position from the checkpoint history
the service sends on each invocation. See sibling SDKs (Python/JS/Java) listed in `README.md` for the shared
model — this SDK deliberately mirrors their semantics.

## Build & test

Targets `net8.0;net10.0` (`DefaultPackageTargets` in `buildtools/common.props`). `TreatWarningsAsErrors` is on
everywhere, and the main library is `IsTrimmable` with the trim analyzer enabled — keep new code AOT/trim-clean.

```bash
# Build the library (run from this directory)
dotnet build

# Unit tests (fast, no AWS). Project: Libraries/test/Amazon.Lambda.DurableExecution.Tests
dotnet test ../../test/Amazon.Lambda.DurableExecution.Tests/Amazon.Lambda.DurableExecution.Tests.csproj

# A single test
dotnet test ../../test/Amazon.Lambda.DurableExecution.Tests/Amazon.Lambda.DurableExecution.Tests.csproj \
  --filter "FullyQualifiedName~StepOperationTests"

# Coverage report (requires reportgenerator tool)
../../test/Amazon.Lambda.DurableExecution.Tests/coverage.sh
```

Unit tests reach `internal` types via `InternalsVisibleTo` (declared in the `.csproj`). They use
`Amazon.Lambda.TestUtilities` (`TestLambdaContext`) and the real `SourceGeneratorLambdaJsonSerializer` —
set `TestLambdaContext.Serializer` so `LambdaSerializerHelper.GetRequired` finds one.

### Integration tests (expensive, real AWS)

`Libraries/test/Amazon.Lambda.DurableExecution.IntegrationTests` deploys real Lambdas. Each test builds a
`TestFunctions/<X>/` project with **`dotnet publish` (framework-dependent, linux-x64)**, zips the publish
output, and deploys it as a **zip package on the managed `dotnet10` runtime** (executable model,
`Handler=bootstrap`) — no Docker or ECR. `DurableFunctionDeployment` creates an IAM role + Lambda (with
`DurableConfig` and JSON `LoggingConfig`), invokes it, and tears everything down on dispose.
Requires only the .NET SDK + AWS creds (us-east-1); no Docker. Slow, but no container build. Every behavior
in `docs/` should have a paired integration test under that project. Prefix AWS commands with
`unset AWS_PROFILE` to use `[default]` creds.

**Run integration tests against `net10.0`.** The project multi-targets `net8.0;net10.0`; `dotnet test`
without a framework spins up one testhost per TFW and runs them concurrently, which races two processes on
the same `TestFunctions/<X>/` build dir. Pin the framework:

```bash
dotnet test ../../test/Amazon.Lambda.DurableExecution.IntegrationTests/Amazon.Lambda.DurableExecution.IntegrationTests.csproj \
  -f net10.0 --filter "FullyQualifiedName~MultipleStepsTest"
```

## Architecture: the replay model

This is the part you must understand before changing anything. Read these together:
`DurableFunction.cs`, `DurableExecutionHandler.cs`, `DurableContext.cs`, `Internal/DurableOperation.cs`,
`Internal/ExecutionState.cs`, `Internal/OperationIdGenerator.cs`, `Internal/TerminationManager.cs`.

**Entry point.** The user's Lambda handler delegates to `DurableFunction.WrapAsync<TInput,TOutput>`, which:
hydrates `ExecutionState` from `invocationInput.InitialExecutionState` (paging the service via `NextMarker`),
extracts the user payload from the `EXECUTION`-type op, builds a `CheckpointBatcher` + `DurableContext`, runs
the workflow through `DurableExecutionHandler.RunAsync`, drains checkpoints, and maps the result to a
`DurableExecutionInvocationOutput` with status **Succeeded / Failed / Pending**.

**Each operation runs the same workflow code every invocation.** There is no persisted program counter.
On re-invocation the user function executes from the top again; each durable call (`StepAsync`, `WaitAsync`,
etc.) looks up its own checkpoint and either replays the cached result or runs fresh. This is why workflow
code **must be deterministic** — same operations, same order, same names across deployments.

**Deterministic operation IDs** (`OperationIdGenerator`). Each durable call gets an ID = SHA-256 of
`"<parentPrefix>-<counter>"`, where the counter is per-context and pre-incremented. The same workflow position
yields the same opaque ID across replays, so a checkpoint correlates to a call by *position*, not by name —
renaming a step does **not** break replay (the human name rides separately on `OperationUpdate.Name`).
Reordering or adding/removing calls *does* break it. `ValidateReplayConsistency` enforces this and throws
`NonDeterministicExecutionException` on type/name drift.

**Suspension is implemented by never completing a Task** (`TerminationManager` + `DurableExecutionHandler`).
When an op must suspend (wait timer, scheduled retry, pending callback/invoke) it calls

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws/aws-lambda-dotnet](https://github.com/aws/aws-lambda-dotnet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
