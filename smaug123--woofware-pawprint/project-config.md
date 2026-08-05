---
trigger: always_on
description: WoofWare.PawPrint is an experimental .NET runtime implementation written in F#. It's an IL interpreter designed to be:
---

# WoofWare.PawPrint

WoofWare.PawPrint is an experimental .NET runtime implementation written in F#. It's an IL interpreter designed to be:
- Fully deterministic (supporting time-travel debugging and fuzzing over thread execution order)
- Fully managed (reimplementing P/Invoke methods to avoid native code)
- Fully in-memory except for explicit filesystem operations

This is NOT a high-performance runtime - it's a very slow IL interpreter prioritizing determinism over speed.

If you need to check upstream behaviour, the genuine .NET runtime's source is pinned in `flake.nix` (`dotnet-runtime-src`) and exposed inside the Nix devshell as `$DOTNET_RUNTIME_SRC`. The pin tracks the .NET 10 servicing version the devshell runs (kept honest by the `runtime-version-pin` flake check and the `TestEmulatedRuntime` drift test). To keep the closure small it is sparse-checked-out to the trees we read most — `src/coreclr`, `src/libraries/System.Private.CoreLib`, and `eng`; if you need another tree, add it to the `sparseCheckout` list in `flake.nix`. See the `.claude/commands/sync-dotnet-runtime.md` Claude command for how to bump the pin. (The old ad-hoc `../dotnet-runtime` sibling checkout is no longer used; `../dotnet`, without the `-runtime` suffix, is the .NET SDK source and is not what you want.)

Standard `dotnet` toolchain is provided by the Nix devshell. Run `dotnet` commands as `nix develop -c dotnet ...` rather than invoking `dotnet` directly.

After changes, `nix develop -c dotnet fantomas .` to format.

The solution file is `WoofWare.PawPrint.slnx` (slnx format).

### Running the Application
A playground C# file is in CSharpExample/Class1.cs.
This environment is convenient for running WoofWare.PawPrint against a standalone DLL.
Interpolate the appropriate platform/config strings as necessary.

```bash
nix develop -c dotnet publish --self-contained --configuration Release --runtime osx-arm64 CSharpExample/
nix develop -c dotnet run --project WoofWare.PawPrint.App/WoofWare.PawPrint.App.fsproj -- CSharpExample/bin/Release/net9.0/osx-arm64/publish/CSharpExample.dll
```

## Architecture

### Core Components

**WoofWare.PawPrint** (Main Library)
- `AbstractMachine.fs`: Core IL interpreter execution engine, knitting together `UnaryConstIlOp.fs`, `UnaryMetadataIlOp.fs`, `UnaryStringTokenIlOp.fs`, and `NullaryIlOp.fs`
- `IlMachineState.fs`: Manages the complete state of the abstract machine
- `MethodState.fs`: Tracks execution state of individual methods
- `ManagedHeap.fs`: Implements the managed memory model
- `Assembly.fs`: Handles reading and parsing .NET assemblies
- `TypeInfo.fs`, `TypeDefn.fs`, `TypeRef.fs`: Type system implementation
- `IlOp.fs`: IL instruction definitions and munging
- `EvalStack.fs`: Evaluation stack implementation
- `Corelib.fs`: Core library type definitions (String, Array, etc.)
- `Native/` (dispatched by `Native/NativeDispatch.fs`) and `ExternImplementations/`: the boundary for runtime-provided or host-provided behavior; prefer extending this seam over special-casing host effects elsewhere in the interpreter
- `EmulatedKernel.fs`: the simulated process's kernel-visible state (virtual clock, seeded PRNG, fd table, env vars, processor count). Values the real runtime would read from the host belong here as *data*, never as a host read: the library must not call `System.Environment`, `DateTime.Now`, `Guid.NewGuid` or similar, because a replay would then depend on the machine that produced it

**WoofWare.PawPrint.Test**
- Uses NUnit as the test framework
- Test cases are defined in `TestPureCases.fs` and `TestImpureCases.fs`
- C# source files in `sources{Pure,Impure}/` are compiled and executed by the runtime as test cases; files in `sourcesPure` are automatically turned into test cases with no further action (see TestPureCases.fs for the mechanism), while `sourcesImpure` tests must be explicitly registered
- The `unimplemented` set of test files that are not yet expected to pass lives in `WoofWare.PawPrint.Test/TestPureCases.fs` (look for `let unimplemented =` near the top of the `TestPureCases` module)
- `TestHarness.fs` provides infrastructure for running test assemblies through the interpreter
- Run all tests with `nix develop -c dotnet test WoofWare.PawPrint.Test/WoofWare.PawPrint.Test.fsproj --verbosity normal`
- Run a filtered subset with `nix develop -c dotnet test WoofWare.PawPrint.Test/WoofWare.PawPrint.Test.fsproj --no-build --filter "Name~TypeRef" --verbosity normal`
- List adapter-discovered tests with `nix develop -c dotnet test WoofWare.PawPrint.Test/WoofWare.PawPrint.Test.fsproj --list-tests`
- The `dotnet run`-based runner (`dotnet run --project ... -- --filter-test-case Foo --no-spinner`) may produce no visible output in non-interactive shells; prefer `dotnet test` with `--filter "Name~..."` instead

**WoofWare.PawPrint.App**
- Entry point application for running the interpreter

**WoofWare.PawPrint.IlDump**
- Small CLI tool for disassembling IL from .NET assemblies, using the same assembly-reading infrastructure as the interpreter
- Usage: `nix develop -c dotnet run --project WoofWare.PawPrint.IlDump -- <dll-path> [TypeName] [MemberName]`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Smaug123/WoofWare.PawPrint](https://github.com/Smaug123/WoofWare.PawPrint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-04 -->
