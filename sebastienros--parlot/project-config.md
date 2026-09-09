---
trigger: always_on
description: Guidance for AI agents working in this repository. This is the single source of truth: `CLAUDE.md` and
---

# AGENTS.md

Guidance for AI agents working in this repository. This is the single source of truth: `CLAUDE.md` and
`.github/copilot-instructions.md` only point here. Record new guidance in this file.

Parlot is a parser combinator library whose reason to exist is speed. Every change is judged on allocations
and throughput first, ergonomics second. Assume any code you touch under `src/Parlot` is on a hot path
until a benchmark says otherwise.

## Build, test, benchmark

The SDK is pinned by `global.json` (10.0.100, `rollForward: latestMajor`). Tests run on
Microsoft.Testing.Platform (also configured in `global.json`) with xunit v3.

```bash
dotnet build                       # all TFMs: net472, netstandard2.0, net8.0, net10.0 (~3s incremental)
dotnet test test/Parlot.Tests/Parlot.Tests.csproj -f net10.0
dotnet test test/Parlot.SourceGenerator.Tests/Parlot.SourceGenerator.Tests.csproj   # net10.0 only
dotnet test test/Parlot.Standalone.Tests/Parlot.Standalone.Tests.csproj              # net8.0/net10.0; net472 on Windows, no Parlot reference
dotnet build test/Parlot.Standalone.NetStandard/Parlot.Standalone.NetStandard.csproj # netstandard2.0 generated consumer
```

Develop and validate against `net10.0` first; only widen to the other TFMs once the behaviour is right.

**`-f net10.0` does not work solution-wide.** `dotnet build -f net10.0` and `dotnet test -f net10.0` from
the root fail with `NETSDK1005`, because `Parlot.SourceGenerator` targets `netstandard2.0` only. Pass `-f`
to an individual project, or build everything without `-f`.

### Running a single test

Microsoft.Testing.Platform has no VSTest `--filter`. Use xunit v3's filters, after `--`:

```bash
dotnet test test/Parlot.Tests/Parlot.Tests.csproj -f net10.0 -- --filter-method "*.ShouldReturnElse*"
```

Faster inner loop — run the test host directly (it is an `Exe`), no MSBuild pass:

```bash
dotnet build test/Parlot.Tests/Parlot.Tests.csproj -f net10.0
test/Parlot.Tests/bin/Debug/net10.0/Parlot.Tests.exe --filter-method "*.ShouldReturnElse*"   # drop .exe on Unix
```

Also available: `--filter-class`, `--filter-namespace`, `--filter-trait`, `--filter-query`,
`--filter-not-*`, `--list-tests`. Simple filters and query filters cannot be mixed.

### Benchmarks

```bash
dotnet run --project test/Parlot.Benchmarks/Parlot.Benchmarks.csproj -c Release -- --list flat
dotnet run --project test/Parlot.Benchmarks/Parlot.Benchmarks.csproj -c Release -- --filter "*Json*"
```

`-p` no longer resolves a project directory; pass `--project` with the full `.csproj` path.

### Two things that will waste your time

- `TreatWarningsAsErrors` is on repo-wide (`Directory.Build.props`) with `AnalysisLevel=latest-Recommended`
  for `src`. An unused variable fails the build.
- `Parlot.Benchmarks` and `Parlot.SourceGenerator.Tests` load `src/Parlot/bin/$(Configuration)/netstandard2.0/Parlot.dll`
  as an `<Analyzer>`, because the generator *executes* your parser code at compile time. After changing
  `src/Parlot`, run a plain `dotnet build` in the **same configuration** before trusting generated output —
  a stale `netstandard2.0` assembly means the generator emits code from the old parser logic, and a missing
  one breaks generation outright.

## Architecture

Two layers, plus a compile-time path that mirrors the runtime one.

**Scanning layer** (`src/Parlot`): `Scanner` owns the input `string` and a `Cursor`; `TextSpan` carries
buffer + offset + length so no substring is ever allocated. `Character` is a partial class split by
technique: `Character.SearchValues.cs` for net8.0+, `Character.Mask.cs` plus the byte table in
`Character.Generated.cs` for everything below. That table is **generated** — don't hand-edit it; rerun
`CanGenerateMasks` in `test/Parlot.Tests/CharMaskGeneratorTest.cs` and take the string it builds from the
debugger.

**Combinator layer** (`src/Parlot/Fluent`): `Parser<T>` is the abstract base; the whole library is
instances of it composed into a graph. `Parsers` is the static entry point exposing the `Literals` and
`Terms` builder structs; the combinators are spread over `Parsers.*.cs` / `ParserExtensions.*.cs` by
concern. `Deferred<T>` closes recursive grammars.

`Parser<T>.Parse(ParseContext, ref ParseResult<T>)` is the hot method and carries a contract
(see `docs/writing.md`, read it before writing a parser):

- bracket the body with `context.EnterParser(this)` / `context.ExitParser(this)`;
- **on failure the cursor must be back where it started** — `Cursor.ResetPosition(start)` when this parser
  advanced it, but not when a sub-parser failed (that one already reset itself);
- write a test that asserts the cursor position is restored on failure.

### The optimization surface — three opt-in interfaces

Most of Parlot's speed comes from parsers advertising capabilities rather than from the `Parse` bodies.
A new parser type should implement each one that applies:

| Interface | Namespace | Effect |
|---|---|---|
| `ISeekable` | `Parlot.Rewriting` | Declares the first chars that can match, so `OneOf` builds a char lookup table, skips branches that cannot match, and hoists the whitespace skip. About two thirds of the parser types implement it. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sebastienros/parlot](https://github.com/sebastienros/parlot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
