---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`src/Partas.Build` is a **library in early construction**. It merges three things that already exist separately in sibling repositories:

- **Fun.Build** (`../Fun.Build`) — the pipeline/stage/step computation-expression DSL and its execution engine. `Types.fs` is a port of Fun.Build's `Types.fs` + `Types.Internal.fs` + `StageContextExtensions.fs` + `PipelineContextExtensions.fs`, restructured into one file and rewritten from members-on-records into `module StageContext` / `module PipelineContext` functions using `voption` instead of `option`.
- **System.CommandLine 2.0.11** via a vendored, adapted copy of `FSharp.SystemCommandLine`'s input layer (`System.CommandLine/Inputs.fs`, `Aliases.fs`) — `ActionInput<'T>`, the `Input.xxx` combinators, `ActionContext`.
- **ActionPath** (from `../Partas.ProjectTemplates`) — the `ActionContext -> ActionContext` composable build step, where each step reads its own flags so commands are flat ordered lists rather than a dependency graph. It is where the flat-ordered-list shape came from; this repo's own `Build/` CLI used it until Phase 7 replaced it with the library.

The goal of the merge: a Fun.Build-style pipeline that *declares the CLI inputs it needs*, with commands deriving their `System.CommandLine` option set from the pipelines they activate — so options, validation and help text are generated from the pipeline definition instead of registered by hand.

Do not treat `../Fun.Build` as a dependency: it is the reference implementation being absorbed and reshaped. Consult it (and its `CLAUDE.md`) when porting a feature; do not copy its record-member style.

## Read `PLAN.md` first

`PLAN.md` is the agreed design for how input binding works and what changes in `Types.fs` and the builders, plus a running record of what each phase actually did and what the compiler proved along the way. All seven phases are implemented. Read the phase statuses before changing a builder — several of them are the reason a member looks the way it does.

The one-line version: input collection is **applicative**, not monadic. An `InputSpec<'T> = { Inputs: ActionInput list; Read: ParseResult -> 'T }` makes the input set readable without a `ParseResult`, which is what breaks the circularity between registering options and binding their values. Stages become pure (no `ActionContext`), binding happens in an `inputs { let! … and! … return … }` CE, and pipelines/commands harvest `.Inputs` upward.

## Commands

Every task goes through the `Build` CLI project, not a script:

```shell
dotnet run --project Build.fsproj -- --help
dotnet run --project Build.fsproj -- build            # restore + build
dotnet run --project Build.fsproj -- test             # build + the three Expecto suites
dotnet run --project Build.fsproj -- test --quick     # skip restores/clean
dotnet run --project Build.fsproj -- publish          # pack + push (--nuget-key, else the `local` feed)
dotnet run --project Build.fsproj -- bump [BUMP] -p <project>...   # rewrite <Version> in a project file
dotnet run --project Build.fsproj -- docs [--watch]   # fsdocs build / live serve
```

Flags sit on the commands whose stages read them, not on the root: `--quick` (skip restores and the clean), `--skip-tests`, `--configuration Debug|Release`. `<command> --help` is generated from those stages and is the authority on which command takes what.

For a single test, run the Expecto suite directly:

```shell
dotnet run --project tests/Partas.Build.Tests -- --filter-test-case "conditions conjoin rather than replace"
dotnet run --project tests/Partas.Build.Tests -- --list-tests
```

Fast inner loop while working on the library only: `dotnet build src/Partas.Build`.

## Current state (verify before assuming)

- Phases 0-7 of `PLAN.md` are done: `dotnet build src/Partas.Build` is clean and `dotnet run --project Build.fsproj -- test` is green (220 Expecto tests across three suites — 80 in `tests/Partas.Build.Tests`, one file per layer, plus `tests/Partas.Build.ExternalAnnotations.Tests` and `tests/Partas.ExternalAnnotations.Tests`). The `Build/` CLI is itself written against the library, so it is the first thing a breaking change breaks.
- The DSL exists end to end: `inputs` (`Builders/Inputs.fs`), `stage` (`Builders/Stage.fs`), `pipeline` (`Builders/Pipeline.fs`), `command`/`rootCommand` (`Builders/Command.fs`). A stage that declares an input turns its pipeline into an `InputSpec<PipelineContext>`, and the command registers whatever those specs declare. Conditions are in `Builders/Conditions.fs` — `whenAll`/`whenAny`/`whenNot`/`whenEnv`/`whenStage` plus the `when'`/`whenEnvVar`/`whenBranch`/`when{Windows,Linux,OSX}` operations on `StageBuilder`. `Builders.fs` is still an empty stub.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shayanhabibi/Partas.Build](https://github.com/shayanhabibi/Partas.Build) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
