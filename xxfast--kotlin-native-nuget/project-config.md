---
trigger: always_on
description: This file describes common issues and pain points that agents might encounter when they work in this project.
---

# Agent Instructions

This file describes common issues and pain points that agents might encounter when they work in this project. 

If you ever encounter an issue specific to agents in the project, please update this file to help prevent future agents from having the same issue.

## Understand the Project Goals

Read [GOALS.md](GOALS.md) before making design decisions.

## Roadmap

Read [ROADMAP.md](ROADMAP.md) to understand the planned features and priorities for the project. This will help you align your work with the overall direction of the project and avoid working on features that are not currently a priority.

## Feature workflow

- Run the [feature-design skill](.claude/skills/feature-design/SKILL.md) to drive the end-to-end TDD feature workflow. It orchestrates the agents below from the main thread.

## Agents

- Delegate to the [research agent](.claude/agents/research.md) to research how a feature should map across the bridge: Kotlin → C# (forward) or C# → Kotlin (reverse, NuGet consumption)
- Delegate to the [csharp-dev agent](.claude/agents/csharp-dev.md) for C# test development
- Delegate to the [kotlin-dev agent](.claude/agents/kotlin-dev.md) for Kotlin implementation
- Delegate to the [refactorer agent](.claude/agents/refactorer.md) for style cleanup and refactoring
- Delegate to the [documenter agent](.claude/agents/documenter.md) to document a shipped feature: the Writerside pages in `docs/topics/`, ROADMAP.md, FEATURES.md, ADR status. Runs in parallel with the refactorer (Markdown vs Kotlin, no file overlap), but only off a main-thread snapshot of the generated `build/` output, since the refactorer's verify cleans `build/` (see the feature-design skill's Step 5)

## Follow Standard Coding Conventions

- For Kotlin, follow standard coding conventions as described here https://kotlinlang.org/docs/coding-conventions.html
- For C#, follow the standard coding conventions as described here https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions

On top of that, we have some additional conventions that are specific to this repository here [STYLE.md](STYLE.md)

## Follow Repository Coding Conventions

- Naming is hard. Use shorten names when applicable and rely on the type to do the heavy lifting.
  - e.g:- `id: SomeId` instead of `someId: SomeId` 
  - e.g:- `fun get(id: SomeId): Something` instead of `fun getSomethingById(someId: SomeId): Something`
  - e.g:- `val something: List<Something>` instead of `val somethingList: List<Something>`
- Deter using scoping functions that introduce indentation (e.g:- `apply`, `with`, `run`)
- Prefer using `if` statements over `when` statements with just two branches
- When handling error states from `Result`, avoid using scoping functions (such as `.onFailure`) that introduce indentation. 
  - Instead, use explicit `if (result.isFailure)` checks with proper logging and error handling.

## Stay In Scope

- Always ensure that your changes are aligned with the scope of the JIRA ticket you are working on.
- Avoid making unrelated changes or improvements that are not directly related to the ticket, as this can make code reviews more difficult and can introduce unintended side effects.
- Don't touch unrelated code or files that are not necessary for the implementation of the ticket - no matter how small and easy it may look

## Keep the C# Test Project Cross-Platform

- `IntegrationTests/IntegrationTests.csproj` pins a `<RuntimeIdentifier>` so MSBuild copies the matching `runtimes/{rid}/native/*` asset next to the test host (a RID-less framework-dependent build does not, causing `DllNotFoundException` on the `[DllImport]` P/Invoke).
- Do **not** hardcode a specific RID (e.g. `win-x64`). Use `$(NETCoreSdkRuntimeIdentifier)` so it resolves to the host platform. CI runs on `macos-latest` (`osx-arm64`); a hardcoded `win-x64` builds fine but aborts at `dotnet test` with `Could not find 'dotnet' host for the 'X64' architecture`.
- Fixture builds give `TestLibrary` and `TestDependency` a new immutable version on every pack and write `build/FixtureVersions.props`. The fixture consumers import that file, so they restore the exact fresh version without clearing NuGet caches or consumer `obj`/`bin`. Use `scripts/verify-fixture-package-versioning.sh` to guard this behaviour.
- This only fixes the repository fixtures. A real package whose contents change without a version change can still be served from NuGet's cache. If that is possible in a manual test, clear its cached package before restoring; for this repository's full integration path, use `scripts/verify.sh`.
- **Never hand-copy files into `~/.nuget/packages/`**, and never hand-patch a generated shim, to skip a rebuild. Agents have done both to iterate faster and it backfires: MSBuild resolves the `<Compile>` item set from the restored package at restore time, so a hand-edited cache produces symptoms that look exactly like real compiler or generator bugs, and you burn hours chasing a bug that does not exist. Always go through `scripts/verify.sh`.

## Don't Leave Gradle Builds Running


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xxfast/kotlin-native-nuget](https://github.com/xxfast/kotlin-native-nuget) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
