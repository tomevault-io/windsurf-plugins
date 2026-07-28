---
trigger: always_on
description: How the decompiler test suite is structured, what each test kind does, and how to add tests.
---

# ICSharpCode.Decompiler.Tests guide

How the decompiler test suite is structured, what each test kind does, and how to add tests.

## The matrix-testing model

Most fixtures run one logical test against a whole matrix of compilers and options: an NUnit
`[Test]` method takes a `CompilerOptions` parameter fed by `[ValueSource]` from static config
arrays declared per runner (`defaultOptions` including mcs, `roslynOnlyOptions`,
`roslyn2OrNewerOptions`, `roslyn3OrNewerOptions`, `roslyn4OrNewerOptions`, each also in a
`...WithNet40Options` variant). One test method therefore becomes 4-26 test cases.

`CompilerOptions` flags (see `Helpers/Tester.cs`) select:
- the compiler: legacy csc (`None`), `UseRoslyn1_3_2`, `UseRoslyn2_10_0`, `UseRoslyn3_11_0`,
  `UseRoslyn4_14_0`, `UseRoslynLatest` (version comes from `RoslynVersion` in
  `Directory.Packages.props`), or `UseMcs2_6_4`/`UseMcs5_23`
- the target: `TargetNet40` (compiles against .NET Framework reference assemblies from the
  `ILSpy-tests` submodule) vs. .NET Core reference packs (net5.0 for Roslyn 3, current preview
  for Roslyn 4/latest)
- codegen options: `Optimize` (`-o+`, defines `OPT`), `UseDebug`, `Force32Bit`, `Library`,
  `GeneratePdb`, `NullableEnable`, `CheckForOverflowUnderflow`, ...

Compiled artifacts are named `<TestName><suffix>.exe|dll` where the suffix encodes the config
(`Tester.GetSuffix`, e.g. `.opt.roslyn3.net40`). By default they land next to the test case
sources; set `TestsAssemblyTempPath` in `DecompilerTests.config.json` to redirect them
(`Helpers/TestsAssemblyOutput.cs`).

First test run on a machine: the `[SetUpFixture]` in `TestTraceListener.cs` calls
`Tester.Initialize()`, which downloads the Roslyn toolsets, vswhere, and the reference-assembly
packs from NuGet (network required; cached under the test output directory afterwards) and
builds the self-contained `ICSharpCode.Decompiler.TestRunner`. Package downloads check the
`ILSpy-tests/nuget` folder first, so the `ILSpy-tests` submodule must be initialized (see the
root `CLAUDE.md` section on the submodule).

## Test kinds

| Kind | Runner / fixture dir (`TestCases/...`) | Pipeline | Compared against |
|---|---|---|---|
| Pretty | `PrettyTestRunner` / `Pretty/*.cs` | compile -> decompile | the test source itself |
| Correctness | `CorrectnessTestRunner` / `Correctness/*.{cs,vb,il}` | compile -> decompile -> recompile -> execute both | runtime output (stdout/stderr/exit code) of original vs. re-compiled |
| ILPretty | `ILPrettyTestRunner` / `ILPretty/*.il` | ilasm -> decompile | sibling `.cs` file |
| Ugly | `UglyTestRunner` / `Ugly/*.cs` | compile -> decompile with sugar settings disabled | sibling `.Expected.cs` file |
| Disassembler | `DisassemblerPrettyTestRunner` / `Disassembler/Pretty/*.il` | ilasm -> disassemble with our `ReflectionDisassembler` | the `.il` source (or `.expected.il`, e.g. `SortedOutput`) |
| VBPretty | `VBPrettyTestRunner` / `VBPretty/*.vb` | vbc -> decompile to C# | sibling `.cs` file |
| PdbGen | `PdbGenerationTestRunner` / `PdbGen/*.cs` | in-proc Roslyn compile (real PDB = oracle), decompile, generate portable PDB with `PortablePdbWriter`, parse both PDBs' sequence-point blobs | the compiler's PDB, projected to the visible breakpoint map (see below) |
| Roundtrip | `RoundtripAssembly` (inputs from `ILSpy-tests/`) | whole-project decompile -> MSBuild rebuild -> run original NUnit tests against the rebuilt assembly | test-run success |
| Unit tests | `TypeSystem/`, `Semantics/`, `Output/`, `Util/`, `DataFlowTest`, `Metadata/`, `ProjectDecompiler/` | plain in-process NUnit | assertions |

## How to add a test

Common to the file-based kinds: every file in the fixture directory must have a matching test
method - each runner has an `AllFilesHaveTests` test that fails otherwise. The method name must
equal the file name (minus extension); it usually just calls the runner's `Run`/`RunForLibrary`
helper, which picks the file via `[CallerMemberName]`.

- **Pretty** (decompiler produces nice code): add `TestCases/Pretty/MyTest.cs` plus a test
  method choosing the narrowest sensible config group (e.g. C# 8 features need
  `roslyn3OrNewerOptions`). The file is simultaneously input and expected output, so write it
  exactly as ILSpy pretty-prints (tabs, `switch {` on the same line, trailing commas, ...).
  Iterate by running the fixture and adjusting the file to the diff.
- **Correctness** (decompiled code behaves identically): add
  `TestCases/Correctness/MyTest.cs` with a `Main` that prints observable state; the harness
  compiles it, decompiles, re-compiles the decompiled output, executes both, and diffs the
  output streams. Roslyn-non-net40 configs execute through
  `ICSharpCode.Decompiler.TestRunner` (an `AssemblyLoadContext` host); other configs run the
  exe directly.
- **Ugly** (output with decompiler features switched off still compiles/behaves): add
  `MyTest.cs` plus the expected decompilation as `MyTest.Expected.cs`.
- **ILPretty / Disassembler**: add a `.il` file (assembled with the NuGet ilasm) and the
  expected `.cs` (`ILPretty`) or rely on round-tripping the `.il` itself (`Disassembler`).
- **VBPretty**: add `MyTest.vb` and the expected C# decompilation `MyTest.cs`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [icsharpcode/ILSpy](https://github.com/icsharpcode/ILSpy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
