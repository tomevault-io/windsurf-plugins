---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Git Workflow

- **NEVER create or switch git branches on your own**, even if an implementation plan file says so. Always ask first and let the user create the branch (or confirm explicitly). Applies to `git checkout -b`, `git switch -c`, `git branch <new>`, and any `git checkout <existing>` that moves away from the currently checked-out branch. Branch setup decides where work lands — that's the user's call.

## Release Notes

- **Reference the GitHub issue or PR in parentheses right after the change-type label when a change is tied to one**, so readers can jump to the context. If a change isn't tracked by an issue, omit the parenthetical.

  ```
  • CHANGED (#47): iOS UpdateHealthData is now atomic (no more delete-then-insert data-loss window).
  • CHANGED (#48): Unified DataOrigin value.
  • FIXED: GetEarliestAccessibleDateTime returns the epoch floor instead of DateTime.MinValue.
  ```

- **Keep entries short and on-point — one line, one sentence, user-visible outcome.** Don't restate internal class names, refactor details, or the diagnosis chain. The reader cares what changed for them, not how it was implemented. Cite the underlying cause only when it's load-bearing for the user (e.g. "off the main thread" matters because consumers wrap calls in `Task.Run`; a class name that nobody outside the lib references does not).

- **Length cap: aim for ~25 words per bullet, hard limit ~40.** If a bullet doesn't fit on one terminal line at typical width, you're saying too much.

- **Smell-checks that mean a bullet is too long:**
  - lists more than two internal symbols / call sites
  - explains a "previously … now …" diagnosis chain instead of just stating the new behaviour
  - cites a numeric threshold, framework limitation, or memory/GC mechanism that the consumer doesn't act on
  - has more than one sentence

  Cut until none of those apply. Lead with the user-visible effect; one short clause after a dash for the mechanism is plenty.

- **Group only user-visible changes.** Skip pure chores, internal refactors, and doc-only commits unless they have an observable effect (e.g. a perf refactor that measurably speeds up a public API does count).

- Applies to `PackageReleaseNotes` in `Maui.Health.csproj` and any standalone changelog.

## Build Commands

```bash
# Build the library (from repo root)
dotnet build src/Maui.Health/Maui.Health.csproj

# Build for specific platform
dotnet build src/Maui.Health/Maui.Health.csproj -f net10.0-android
dotnet build src/Maui.Health/Maui.Health.csproj -f net10.0-ios

# Build the demo app
dotnet build demo/DemoApp/DemoApp/DemoApp.csproj
```

## Architecture

### Platform Abstraction via Partial Classes

The codebase uses C# partial classes to separate platform-specific code:

- **HealthService**: Core service with partial method declarations
  - `Services/HealthService.cs` - shared interface + partial signatures
  - `Platforms/Android/HealthServiceAndroid.cs` - Android Health Connect implementation
  - `Platforms/iOS/HealthServiceiOS.cs` - iOS HealthKit implementation
  - `Platforms/MacCatalyst/` and `Platforms/Windows/` - stub implementations

- **HealthWorkoutService**: Same pattern for workout/session management
  - Base class contains shared logic (duplicate detection, session persistence via Preferences)
  - Platform implementations handle actual health store operations

### Extension Methods for DTO Mapping

Platform-specific conversions use extension methods in `Platforms/{Platform}/Extensions/`:
- `HealthRecordExtensions.cs` (Android) - converts Java health records to/from DTOs
- `HKQuantitySampleExtensions.cs` (iOS) - converts HKQuantitySample to/from DTOs
- Type dispatch via switch expressions on `typeof(TDto).Name`

### DTO Structure

- `HealthMetricBase` - abstract base with Id, DataOrigin, Timestamp
- Metric DTOs (StepsDto, WeightDto, etc.) inherit from HealthMetricBase
- `WorkoutDto` - standalone workout model with activity type, time range, energy/distance
- `WorkoutSession` - in-memory model for active sessions with pause/resume state

### Key Services

- `IHealthService` - generic health data API: `GetHealthData<TDto>()`, `WriteHealthData<TDto>()`, permissions
- `IHealthWorkoutService` (via `IHealthService.Activity`) - workout CRUD + session management

## Test Commands

```bash
# Run all tests
dotnet test tests/Maui.Health.Tests/Maui.Health.Tests.csproj
```

## Test Conventions

- Test projects go in `tests/` directory and are nested under a `Tests` solution folder in the .sln
- Use xUnit
- Test method names use three-part underscore format: `MethodUnderTest_Scenario_ExpectedResult`
- Use `// Arrange`, `// Act`, `// Assert` comments in every test

### Why tests use `<Compile Include>` instead of `<ProjectReference>`

`Maui.Health.csproj` targets MAUI TFMs (`net10.0-android`, `-ios`, `-maccatalyst`, `-windows`). `Maui.Health.Tests.csproj` targets plain `net9.0` because xUnit runners don't run under MAUI TFMs. You can't `<ProjectReference>` across that boundary, so the test csproj pulls in individual platform-agnostic source files via `<Compile Include="..\..\src\Maui.Health\..." Link="..." />`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kebechet/Maui.Health](https://github.com/Kebechet/Maui.Health) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
