---
trigger: always_on
description: <!-- DO NOT EDIT. Generated mirror of /AGENTS.md. Edit AGENTS.md and run: pwsh tools/Validate-AgentFiles.ps1 -Fix -->
---

<!-- DO NOT EDIT. Generated mirror of /AGENTS.md. Edit AGENTS.md and run: pwsh tools/Validate-AgentFiles.ps1 -Fix -->
# AGENTS.md

Instructions for AI coding agents working in this repository. Applies to GitHub Copilot
(VS Code, Visual Studio, CLI, github.com cloud agent), Claude Code, OpenAI Codex, Cursor,
Aider, Gemini CLI, and any other tool that supports the [AGENTS.md](https://agents.md/)
standard.

This file is the single source of truth. `.github/copilot-instructions.md` mirrors this
file for Copilot features that read it directly; do not edit the mirror by hand.

For broader contributor guidance, see [CONTRIBUTING.md](../CONTRIBUTING.md) and
[docs/coding_guidelines.md](../docs/coding_guidelines.md). For how to add or update agent
customizations (skills, prompts, custom agents, path-specific instructions), see
[docs/agent-customization.md](../docs/agent-customization.md).

## Project overview

`touki` is a C# library that targets both **.NET 10** and **.NET Framework 4.7.2**.
All code must compile and behave correctly on both targets.

Top-level layout:

- `touki/` - main library
- `touki.tests/` - xUnit tests (uses FluentAssertions; access to internals via `InternalsVisibleTo`)
- `touki.testsupport/` - shared test helpers (`TestAccessor`, etc.)
- `touki.perf/` - BenchmarkDotNet performance projects
- `sample/` - usage samples
- `docs/` - contributor and design documentation

## Environment setup

- On Unix, run `./setup.sh` once after cloning to install the .NET 10 SDK and update PATH.
- On Windows, install the .NET SDK from <https://dotnet.microsoft.com/download> and use
  `dotnet` directly. PowerShell is the preferred terminal.
- Use the `dotnet` CLI for building and testing. CI runs `dotnet build` and `dotnet test`.

## Coding style

- Use the latest C# features (C# 14) where applicable.
- Always use C# keywords for types (`int`, `string`, `bool`) instead of their aliases
  (`Int32`, `String`, `Boolean`).
- Always use `nint` and `nuint` for native integer types (not `IntPtr` and `UIntPtr`).
- Avoid `var` - use explicit type declarations, target-typed `new`, and
  [collection expressions](https://learn.microsoft.com/dotnet/csharp/language-reference/operators/collection-expressions)
  together: `List<string> list = new();`, `int[] values = [1, 2, 3];`,
  `Point[] points = [new(1, 2), new(5, 6)];`. The variable's type is always
  spelled out; the `new`/`[]` literal supplies the value.
- Use `is null` and `is not null` for null checks instead of `== null` or `!= null`.
- For enums wrapped in `Value`, always call `Value.Create()` instead of `new Value()`.
- Use the following header for all C# files:

```c#
// Copyright (c) 2025 Jeremy W Kuhne
// SPDX-License-Identifier: MIT
// See LICENSE file in the project root for full license information
```

## Comments and XML documentation

- Avoid putting comments at the end of lines.
- Comments should be before the code they describe, or inside blocks to describe the
  condition the block is handling.
- Create XML comment documentation for all public methods, properties, and types.
- Use `<para>` blocks in `<remarks>` to separate different sections of remarks.
- Indent XML comments by one space for each level of nesting:

```c#
/// <summary>
///  This is the summary.
/// </summary>
/// <remarks>
///  <para>
///   This is a remark.
///  </para>
/// </remarks>
```

- Use `<inheritdoc/>` and `<inheritdoc cref="..."/>` to inherit documentation from base
  classes and interfaces where applicable.
- For method overloads, use `<inheritdoc cref="MethodName"/>` to inherit documentation
  from the method with the most arguments, overriding tags where they differ.
- Use `<see langword="..."/>` tags for language keywords in comments
  (e.g. `<see langword="true"/>` instead of `true`).
- Prefer plain ASCII characters that don't need escaping (`-`, `"`, `...`,
  `->`) over typographic Unicode (`—`, `–`, `"`, `"`, `…`, `→`) or HTML
  entities (`&mdash;`, `&ndash;`, `&hellip;`, `&rarr;`, `&nbsp;`). Use a
  plain `-` (with surrounding spaces when separating clauses) instead of
  em/en-dashes. Only escape when the raw character would actually be
  parsed as markup (e.g. `<` inside an XML doc comment); do not escape
  defensively when it isn't needed. Exact vendored payloads are exempt: fix
  generic punctuation upstream and re-vendor rather than editing a pinned core.

## Line breaks and whitespace

- Never put multiple statements on a single line.
- Ensure there is always a single blank line between methods and properties.
- Preserve existing spaces and line breaks when making edits, except when fixing
  whitespace issues.
- Indents are 4 spaces for all code except for XML (including XML comments in sources),
  which should have nested tags and content indented by one space per level.
- Lines should never end in or contain only spaces or tabs.
- Lines should be broken before 120 characters if they would otherwise exceed 150
  characters.
- When breaking statements, the next lines should be indented.
- When breaking statements, operators (`+`, `-`, `&&`, `||`, `or`, `and`) should not be
  at the end of the previous line, except for `=>`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JeremyKuhne/touki](https://github.com/JeremyKuhne/touki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
