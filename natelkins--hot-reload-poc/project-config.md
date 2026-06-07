---
trigger: always_on
description: The primary solution sits at `hot_reload_poc/HotReloadPoc.sln`, grouping the F# demo in `hot_reload_poc/src/TestApp/` and the comparative C# harness in `hot_reload_poc/src/csharp_delta_test/`. Place new runtime experiments alongside these directories to keep the solution tidy. Use the `notes/` folder for deep dives on metadata formats or architectural context, and leave generated `bin/` and `obj/` outputs untracked. Helper assets such as `analyze_il.sh`, `setup.fsx`, and the Roslyn playground un
---

# Repository Guidelines

## Project Structure & Module Organization
The primary solution sits at `hot_reload_poc/HotReloadPoc.sln`, grouping the F# demo in `hot_reload_poc/src/TestApp/` and the comparative C# harness in `hot_reload_poc/src/csharp_delta_test/`. Place new runtime experiments alongside these directories to keep the solution tidy. Use the `notes/` folder for deep dives on metadata formats or architectural context, and leave generated `bin/` and `obj/` outputs untracked. Helper assets such as `analyze_il.sh`, `setup.fsx`, and the Roslyn playground under `roslyn/` exist to inspect or prototype IL changes—keep scripts there so other agents can locate them quickly.
- Mirror Roslyn hot reload patterns wherever possible: reuse its terminology, metadata structures (EmitBaseline, DefinitionMap, SymbolMatcher, DeltaMetadataWriter), and component layout when adding new F# functionality so parity stays obvious.
- When working on hot-reload metadata or runtime integration, treat Roslyn as the source of truth. Before committing, diff your changes against the equivalent Roslyn files (`MetadataWriter.cs`, `MetadataSizes.cs`, `ApplyDelta.cs`, etc.) and verify byte-for-byte behavior via both mdv and `MetadataUpdater.ApplyUpdate`. If either parity check fails, the change is not “done.” Capture the Roslyn commit/date you mirrored in your PR/commit description so the next agent can pick up where you left off.

## Build, Run, and Hot Reload Commands
- `dotnet build hot_reload_poc/HotReloadPoc.sln` compiles every project for .NET 9; pass `-c Release` when profiling perf-sensitive code paths.
- `DOTNET_MODIFIABLE_ASSEMBLIES=debug dotnet run --project hot_reload_poc/src/TestApp/TestApp.fsproj` launches the F# hot reload demo and streams delta-generation diagnostics.
- `dotnet run --project ../hotreload-utils/src/hotreload-delta-gen/src/hotreload-delta-gen.csproj -- -msbuild:hot_reload_poc/src/csharp_delta_test/csharp_delta_test.csproj -script:hot_reload_poc/src/csharp_delta_test/diffscript.json` reproduces the C# delta workflow once `hotreload-utils` is cloned adjacent to this repo.
- `./hot_reload_poc/analyze_il.sh <baseline.dll> <patched.dll>` exports IL with `ilspycmd` and highlights differences for manual review.
- `dotnet run --project tools/fsc-watch/fsc_watch.fsproj -- ...` starts a long-lived watch loop; never launch it on behalf of the user without confirming because it deliberately stays running until the user edits a file or presses Ctrl+C.

## Coding Style & Naming Conventions
Follow the existing F# style: four-space indentation, module-qualified functions, and `CamelCase` modules or types. Favor pipeline-friendly expressions over nested calls, and keep reflection helpers in separate modules (see `DeltaGenerator.fs`). For the C# samples, stick to `PascalCase` types, camelCase locals, and 120-character lines. Run `dotnet format` on touched projects before submitting; it keeps both F# and C# sources aligned with SDK defaults.
- When adding new public or internal types/functions, include XML documentation comments so downstream agents understand their responsibilities without re-reading the implementation.
- Leave short inline comments whenever you touch tricky hot-reload logic (EncLog bookkeeping, heap offsets, etc.) so the next agent understands the intent without re-deriving Roslyn parity from scratch.
- When implementing new behavior, drop a quick comment describing **why** the code is structured that way; future agents should never have to guess at the rationale while following along.

## Testing Guidelines
There is no dedicated test project yet, so treat the console output from `HotReloadTest` as the regression gate: confirm the baseline compilation, delta application, and re-invocation sequences all succeed. When introducing new patch scenarios, add deterministic checks (e.g., validating `SimpleLib.GetValue()` responses) and capture expected console markers in comments. If you add a formal test suite, wire it up via `Microsoft.NET.Test.Sdk` and expose it through `dotnet test` so CI can exercise it uniformly.

## Commit & Pull Request Guidelines
Keep commits focused and descriptive in sentence case (see `git log --oneline` for examples). Reference related issues in the subject or body, and include before/after output snippets for hot reload flows when relevant. Pull requests should describe the scenario exercised, note any required environment variables (such as `DOTNET_MODIFIABLE_ASSEMBLIES`), and attach console excerpts or IL diffs that prove the change works. Request reviewers from agents who touched the same module lately to maintain shared context.
- Whenever a test suite passes (even locally), capture that state in a commit so future iterations can bisect or revert cleanly—don’t keep green builds uncommitted.

## Documentation Updates

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NatElkins/hot_reload_poc](https://github.com/NatElkins/hot_reload_poc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-07 -->
