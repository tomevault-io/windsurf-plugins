---
trigger: always_on
description: Instructions for GitHub Copilot and other AI coding agents working with the .NET SDK repository.
---

# Agent Instructions

Instructions for GitHub Copilot and other AI coding agents working with the .NET SDK repository.

## Overview

This is `dotnet/sdk`, the repository for the core of the .NET SDK. It produces the
`dotnet` CLI driver and the MSBuild tasks, targets, templates, and resolvers that are
shared between the .NET CLI and Visual Studio. The build output is a complete, runnable
`dotnet` installation laid down at `artifacts/bin/redist/<configuration>/dotnet`
(`Debug` by default).

For a high-level project description, build status, and contribution flow, see the
[README](../README.md). For the canonical build/test/debug walkthrough, see the
[Developer Guide](../documentation/project-docs/developer-guide.md).

### Grounding architecture and product claims

Treat this overview as an index, not as independent evidence. In plans, reviews,
root-cause analyses, and AI-facing documentation:

- Verify important architecture, product behavior, and ownership claims against primary
  sources and link the nearest evidence: code or project files for current behavior, and an
  ADR, design document, issue, or PR for decisions and history.
- Prefer repository-relative links for in-repo evidence and link the narrowest durable
  source. Do not cite this overview to support itself.
- When a cited in-repo source is relevant to the task, inspect it before relying on the
  claim; do not assume the link target's contents are already in context.
- Identify inference explicitly and cite its inputs. If sources disagree or evidence is
  incomplete, state the uncertainty instead of turning synthesis into fact; update stale
  context in the same change.

### What the SDK does

- Provides the `dotnet` command-line driver (`dotnet build`, `restore`, `publish`, `test`,
  `run`, `watch`, etc.); see the [managed entry point](../src/Cli/dotnet/Program.cs) and
  [registered command tree](../src/Cli/dotnet/Parser.cs).
- Ships the MSBuild logic that turns a `.csproj`/`.fsproj`/`.vbproj` into a build; see the
  [SDK entry points](../src/Tasks/Microsoft.NET.Build.Tasks/sdk/) and
  [tasks and targets](../src/Tasks/Microsoft.NET.Build.Tasks/).
- Bundles related toolsets: project/item templates, Razor/Blazor/Web/Static Web Assets
  SDKs, container publishing, file/format/watch tools, API compatibility tooling, and
  workload management. The final SDK layout is assembled by the
  [`redist` project](../src/Layout/redist/redist.csproj) and its
  [layout targets](../src/Layout/redist/targets/Directory.Build.targets); in-box project
  and item template sources live in [`template_feed`](../template_feed/).

### Repository boundaries and the VMR

An SDK command or build can expose behavior implemented by another .NET repository. Find
the component that defines the behavior before making a change; do not add an SDK
workaround merely because the symptom appears through `dotnet`.

| Repository | Ownership boundary |
| --- | --- |
| [`dotnet/runtime`](https://github.com/dotnet/runtime) | CLR and Mono, the base class libraries, the native `dotnet` host/muxer and apphost, runtime and reference packs, and runtime-owned deployment tooling such as NativeAOT and ILLink. SDK publish targets integrate with these artifacts but do not own their implementation. |
| [`dotnet/roslyn`](https://github.com/dotnet/roslyn) | The C# and Visual Basic compilers, compiler server, compiler APIs, and C#/VB compiler behavior such as language diagnostics and code generation. The SDK supplies inputs and ships Roslyn artifacts; SDK-generated defaults and command wiring remain SDK-owned. |
| [`dotnet/fsharp`](https://github.com/dotnet/fsharp) | The F# compiler and F#-specific tooling. |
| [`dotnet/msbuild`](https://github.com/dotnet/msbuild) | The MSBuild engine, evaluation and execution semantics, logging, and core tasks and targets. SDK-specific `Microsoft.NET.*` tasks and targets remain in this repo. |
| [`NuGet/NuGet.Client`](https://github.com/NuGet/NuGet.Client) | NuGet restore, package resolution, protocols, and related MSBuild tasks. SDK CLI wrappers and SDK-specific integration remain in this repo. |
| [`dotnet/project-system`](https://github.com/dotnet/project-system) | Visual Studio-specific project-system behavior. |
| [`dotnet/dotnet`](https://github.com/dotnet/dotnet) | The Virtual Monolithic Repository (VMR): a synchronized mirror of product repositories plus the infrastructure for building and servicing the integrated .NET product. Product source is mirrored under `src/<repo>`; normal component development still belongs in the owning product repository. |

Do not infer ownership from a diagnostic ID or generated code alone. C# and Visual Basic
compiler diagnostics and compiler-emitted code belong to Roslyn, but analyzers and source
generators belong to the repository that implements them, such as `dotnet/runtime` for
runtime-library generators or `dotnet/sdk` for SDK analyzers.

### Architecture and major components

The managed CLI dispatches commands registered in
[`Parser.cs`](../src/Cli/dotnet/Parser.cs). Unmatched input goes through external command
resolution and then file-based app fallback, as implemented by
[`Program.cs`](../src/Cli/dotnet/Program.cs).

Major source areas under [`src/`](../src/):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dotnet/sdk](https://github.com/dotnet/sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
