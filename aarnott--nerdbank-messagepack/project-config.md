---
trigger: always_on
description: **ALWAYS follow these instructions first and only fallback to additional search and context gathering if the information here is incomplete or found to be in error.**
---

# Copilot instructions for this repository

**ALWAYS follow these instructions first and only fallback to additional search and context gathering if the information here is incomplete or found to be in error.**

* Review the `CONTRIBUTING.md` file for instructions to build and test the software.
* Run the `.github/Prime-ForCopilot.ps1` script (once) before running any `dotnet` or `msbuild` commands.
  If you see any build errors about not finding git objects or a shallow clone, it may be time to run this script again.

## Working Effectively

### Bootstrap and Build
**CRITICAL**: Set the `NBGV_GitEngine` environment variable to `Disabled` before running ANY `dotnet` or `msbuild` commands.

```bash
export NBGV_GitEngine=Disabled
```

**Setup dependencies** (takes ~2-3 seconds):
```bash
./init.ps1 -UpgradePrerequisites -NoNuGetCredProvider
```

**Build the repository** (takes 7-76 seconds depending on cache - do not cancel unless it exceeds 10-15 minutes, set timeout to 10-15 minutes):
```bash
dotnet build tools/dirs.proj -t:build,pack --no-restore -c Release
```

### Testing
**Run tests** (takes ~25 seconds - NEVER CANCEL, set timeout to 5-10 minutes):
```bash
dotnet test --no-build -c Release -- --filter-not-trait "TestCategory=FailsInCloudTest"
```

### Code Quality
**Verify code formatting** (takes ~71 seconds - NEVER CANCEL, set timeout to 90+ minutes):
```bash
dotnet format --verify-no-changes --no-restore
```

**Build documentation** (takes ~19 seconds):
```bash
DocFx=true dotnet docfx docfx/docfx.json --warningsAsErrors --disableGitFeatures
```

**NEVER CANCEL**: Code formatting verification takes approximately 71 seconds. This is normal and expected.

## Validation Scenarios
**ALWAYS test functionality after making changes by running validation scenarios:**

**Test AOT Native Console sample**:
```bash
cd test/AotNativeConsole
dotnet run --no-build -c Release
```
Expected output: JSON data followed by tree structure with fruits and seeds, ending with "Success".

**Test ASP.NET MVC integration**:
```bash
cd samples/AspNetMvc
dotnet run --no-build -c Release
```
Should start web server without errors (web UI testing limited in this environment).

## Performance optimization

* Establish a BenchmarkDotNet baseline before changing a hot path. For primitive integer encoding and decoding, run:
  ```bash
  dotnet run --project test/Benchmarks/Benchmarks.csproj -c Release -f net10.0 -- --filter "*IntegerPrimitives*" --job short
  ```
* Keep benchmark input distributions explicit and reproducible. `Small`, `Mixed`, and `Large` integer datasets exercise distinct MessagePack encodings and branch-prediction behavior; do not replace them with a single representative input.
* For branch-sensitive work, use sufficiently large randomized datasets so a branch predictor cannot learn a short repeating sequence. Preserve the fixed random seed unless intentionally changing the workload.
* Review allocation, generated assembly, branch instructions, and branch mispredictions alongside elapsed time. Hardware counters require an elevated Windows process; an unavailable counter is not evidence of zero misses.
* Benchmark changes measure behavior; they do not prove correctness. Verify all MessagePack encoding boundaries and error behavior with the relevant tests before accepting an optimization.
* Prefer narrowly targeted candidates and retain a simple, verified baseline until benchmark results and generated assembly demonstrate a repeatable improvement for the intended distributions.

## Repository Structure

### Key Projects (src/)
- `Nerdbank.MessagePack` - Main MessagePack serialization library
- `Nerdbank.MessagePack.SignalR` - SignalR integration
- `Nerdbank.MessagePack.AspNetCoreMvcFormatter` - ASP.NET Core MVC formatter
- `Nerdbank.MessagePack.Analyzers` - Roslyn analyzers and code fixes

### Test Projects (test/)
- Each shipping project has a corresponding `.Tests` project
- `AotNativeConsole` - NativeAOT compatibility validation
- `Benchmarks` - Performance benchmarks

### Samples (samples/)
- `AspNetMvc` - ASP.NET Core MVC integration example
- `SignalR` - SignalR integration example
- `cs` and `fs` - C# and F# usage examples

## Software Design

* Design APIs to be highly testable, and all functionality should be tested.
* Avoid introducing binary breaking changes in public APIs of projects under `src` unless their project files have `IsPackable` set to `false`.

## Testing

**IMPORTANT**: This repository uses Microsoft.Testing.Platform (MTP v2) with xunit v3. Traditional `--filter` syntax does NOT work. Use the options below instead.

* There should generally be one test project (under the `test` directory) per shipping project (under the `src` directory). Test projects are named after the project being tested with a `.Tests` suffix.
* Tests use xunit v3 with Microsoft.Testing.Platform (MTP v2). Traditional VSTest `--filter` syntax does NOT work.
* Some tests are known to be unstable. When running tests, you should skip the unstable ones by using `-- --filter-not-trait "TestCategory=FailsInCloudTest"`.

### Running Tests

**Run all tests**:
```bash
dotnet test --no-build -c Release
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AArnott/Nerdbank.MessagePack](https://github.com/AArnott/Nerdbank.MessagePack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
