---
trigger: always_on
description: - **Always run `just test` (unit + integration) to verify changes** — running only `just test-unit` is not acceptable. Unit tests prove nothing about the wider system; integration tests exercise Docker, container startup, bind mounts, and the full Aspire lifecycle.
---

# Copilot Instructions

## Non-negotiables

- **Always run `just test` (unit + integration) to verify changes** — running only `just test-unit` is not acceptable. Unit tests prove nothing about the wider system; integration tests exercise Docker, container startup, bind mounts, and the full Aspire lifecycle.
- **Never claim a fix or task is complete unless ALL tests have passed** — every single test, not just those you believe are related to your change. All tests always are. A task is not done, a fix is not done, nothing is done until `just test` exits green with zero failures.
- **Never skip tests without explicit user permission** — do not use `--filter`, `[Skip]`, or any other mechanism to exclude tests unless the user has explicitly said so for that specific case.
- **Always run `just lintcheck` (or `just lintfix`) before committing** — CSharpier formatting is enforced and CI will reject unformatted code.
- **Never add `Co-authored-by` trailers to commit messages** — do not include Copilot or any other co-author attribution.

## Build, test, and lint

This repo uses [just](https://just.systems/) as a task runner. All commands operate on `src/AspireC4.slnx`.

```sh
just restore          # Restore NuGet packages and local tools
just build            # Build in Release (default)
just build Debug      # Build in Debug

just test             # Run all tests (unit + integration)
just test-unit        # Run unit tests only
just test-integration # Run integration tests only

just lintcheck        # Check formatting with CSharpier
just lintfix          # Auto-fix formatting with CSharpier
```

**Running a single test** — use `--filter` with the Microsoft.Testing.Platform style:
```sh
dotnet test src/tests/AspireC4.UnitTests/AspireC4.UnitTests.csproj -- --filter "FullyQualifiedName~Generate_EmptyModel"
dotnet test src/tests/AspireC4.IntegrationTests/AspireC4.IntegrationTests.csproj -- --filter "FullyQualifiedName~MethodName"
```

Integration tests require Docker to be running and pull `ghcr.io/likec4/likec4`.

## Architecture

This is a .NET Aspire extension library that auto-generates live [LikeC4](https://likec4.dev) architecture diagrams from the Aspire resource graph.

### Projects

| Project | Purpose |
|---|---|
| `src/src/AspireC4` | Aspire integration: lifecycle hook, Docker/CLI server resources, dashboard integration |
| `src/src/AspireC4.Core` | Pure model & DSL: `LikeC4ModelBuilder`, `LikeC4DslGenerator`, annotations |
| `src/tests/AspireC4.UnitTests` | Unit tests for model builder, DSL generator, annotations |
| `src/tests/AspireC4.IntegrationTests` | Integration tests using `DistributedApplication` (requires Docker) |
| `src/tests/AspireC4.TestAppHost` | AppHost used by integration tests |

### Data flow

1. `AddAspireC4()` registers the lifecycle hook and a `LikeC4ServerResource` (Docker container: `ghcr.io/likec4/likec4`).
2. On `BeforeStartEvent`, the lifecycle hook calls `LikeC4ModelBuilder.Build()` to traverse the Aspire resource graph into a `LikeC4Model`, then `LikeC4DslGenerator.Generate()` to write `./likec4/gen/model.gen.c4` by default.
3. The LikeC4 container mounts the output directory via a named Docker volume and hot-reloads when the file changes.
4. A debounced background watcher calls `ResourceNotificationService` to detect state changes and regenerate the file, updating element colours to reflect live resource states.
5. **Windows HMR note**: the HMR endpoint stays on fixed host port `24678`, and Docker Desktop uses polling-based file watching because host filesystem events do not propagate reliably into the container.

**Alternative server mode**: `.WithLocalCLI()` swaps the Docker container for a local Node.js CLI (npx/pnpm/yarn/bun/deno). `.WithHideFromDashboard()` removes the sidecar from the Aspire dashboard and surfaces a link+command on each `ProjectResource` instead.

### Exclusion logic

Resources are excluded from the diagram if they carry `ExcludeFromLikeC4Annotation` or if their `ResourceSnapshotAnnotation.InitialSnapshot.IsHidden == true`. The LikeC4 sidecar resource itself is always excluded.

## Key conventions

### Namespace placement
Public extension methods live in `namespace Aspire.Hosting` (matching Aspire's own namespace) and require suppressing the IDE0130 warning:
```csharp
#pragma warning disable IDE0130
namespace Aspire.Hosting;
#pragma warning restore IDE0130
```
Internal implementation classes use `namespace Aspire.Hosting.AspireC4`.

### Test framework
Tests use **TUnit** (not xUnit/MSTest/NUnit). Key patterns:
```csharp
[Test]
public async Task SomeTest()
{
    await Assert.That(value).IsEqualTo(expected);
}

[Before(Test)]
public async Task SetUpAsync() { ... }
```
Mocking uses **NSubstitute**. Both are globally imported in all test projects via `Directory.Build.props`.

## Test authoring requirements

The following test authoring rules are strict and non-negotiable. All generated tests must comply exactly.

### Required test structure
- Every test must use the AAA pattern.
- Every test must include explicit section comments:
  - `// Arrange`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [purview-dev/aspirec4](https://github.com/purview-dev/aspirec4) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
