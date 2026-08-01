---
trigger: always_on
description: This document captures project-specific knowledge needed to build, test, and evolve Mockly.Http efficiently. It assumes familiarity with .NET, xUnit, and FluentAssertions.
---

### Mockly.Http — Developer Guidelines

#### Scope
This document captures project-specific knowledge needed to build, test, and evolve Mockly.Http efficiently. It assumes familiarity with .NET, xUnit, and FluentAssertions.

---

### Build and Configuration

- Toolchain
  - .NET SDK: The repo uses `global.json`; current CI/runtime targets include .NET 8 and older TFMs. Local SDK 8.x is sufficient; older TFMs are built via multi-targeting.
  - C# language version is enforced via `Directory.Build.props` (`LangVersion` 11). Treat warnings as errors is enabled; analyzers run only for `net8.0` to keep builds fast.

- Solution layout (major parts)
  - `Mockly.Http` — main library (multi-targeted; consumed by tests and FA integration).
  - `FluentAssertions.Mockly.Http.v8` — FluentAssertions extensions for this library.
  - `Mockly.Http.Specs` — xUnit specification-style tests targeting `net6.0` and `net472`.
  - `Mockly.Http.ApiVerificationTests` — Public API approval tests using `PublicApiGenerator` + `Verify`.

- Fast local build
  - Use the Nuke build entrypoint for a consistent environment:
    - PowerShell: `./build.ps1` (auto-installs matching SDK if needed)
    - Bash: `./build.sh`
  - Or build directly with dotnet from the solution root:
    - `dotnet build Mockly.Http.sln -c Debug`

- CI-specific behaviors you may notice locally
  - API verification tests load the compiled `Mockly.Http.dll` for each target framework discovered in `Mockly.Http.csproj` and compare it against the baselined approved API files in `Mockly.Http.ApiVerificationTests/ApprovedApi`. When those tests fail, you need to run the contents of AcceptApiChanges.ps1 or AcceptApiChanges.sh to update the approved files.
  - Analyzer intensity is highest for `net8.0` builds; non-`net8.0` targets suppress analyzers to optimize time.

---

### Testing

- Test projects and frameworks
  - Specs project: `Mockly.Http.Specs` targets `net6.0` and `net472`.
    - Packages: `xunit` 2.5, `xunit.runner.visualstudio` 3.0, `FluentAssertions` 8.8, `FluentAssertions.Web.v8`, `coverlet.collector` for coverage.
  - API approval: `Mockly.Http.ApiVerificationTests` uses `PublicApiGenerator` and `VerifyXunit`. It reads the main library’s target frameworks dynamically.

- Run all tests
  - From the solution root:
    - `dotnet test -c Debug`
  - Using Rider/ReSharper, run the solution or individual projects as usual. Both `net6.0` and `net472` test TFMs are executed.

- Filter tests
  - By fully-qualified name:
    - `dotnet test --filter FullyQualifiedName=Mockly.Http.Specs.MocklyHttpSpecs+BasicUsage.Can_mock_delete_request`
  - By class:
    - `dotnet test --filter FullyQualifiedName~Mockly.Http.Specs.MocklyHttpSpecs+AdvancedMatching`
  - By trait (if you add `[Trait("Category","…")]`):
    - `dotnet test --filter TestCategory=Slow`

- Adding new tests
  - Place new spec classes under `Mockly.Http.Specs` and use xUnit attributes (`[Fact]`, `[Theory]`). Follow the nested-class structure used in `MocklyHttpSpecs` to group scenarios, e.g. `class WhenUsingAssertions`.
  - Prefer FluentAssertions for assertions. For HTTP-specific assertions, use the FA integration in `FluentAssertions.Mockly.Http.v8`.
  - If the test depends on matching HTTP requests:
    - Use `RequestMock` and the fluent builders (e.g., `mock.ForGet().ForPath("/api/*").RespondsWithJsonContent(...)`).
    - Consider `RequestCollection` to capture and assert on requests observed during the test.

- Running API verification tests
  - These tests live in `Mockly.Http.ApiVerificationTests` and ensure the public API of `Mockly.Http` doesn’t change unexpectedly.
  - To re-baseline intentionally changed APIs, run the test once, inspect the diff, and then update approved files in `Mockly.Http.ApiVerificationTests/ApprovedApi`. Use PR review to confirm intended changes.

- Coverage (optional)
  - `coverlet.collector` is already referenced. You can gather coverage via:
    - `dotnet test -c Debug /p:CollectCoverage=true /p:CoverletOutputFormat=cobertura`

---

### Verified demo: adding and running a test

The following minimal test can be temporarily added to `Mockly.Http.Specs` to validate your environment and demonstrate filtering. We verified this flow locally and removed the file afterward to keep the repo clean.

1) Create `Mockly.Http.Specs/DemoEnvironmentSpecs.cs`:
```
using Xunit;

namespace Mockly.Http.Specs;

public class DemoEnvironmentSpecs
{
    [Fact]
    public void Addition_works()
        => Assert.Equal(2, 1 + 1);
}
```

2) Run only this test by FQN:
```
dotnet test --filter FullyQualifiedName=Mockly.Http.Specs.DemoEnvironmentSpecs.Addition_works
```

3) Run the full solution tests:
```
dotnet test
```

4) Delete `DemoEnvironmentSpecs.cs` once done.

---

### Development Notes & Conventions

- Code style and analyzers
  - The repo enforces warnings-as-errors and uses multiple analyzer packages for `net8.0`. Keep code clean to pass builds; fix or suppress with clear justification.
  - Prefer expression-bodied members for small helpers when consistent with surrounding code. Match existing naming and layout patterns.

- Public surface discipline

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dennisdoomen/mockly](https://github.com/dennisdoomen/mockly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
