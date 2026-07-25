---
trigger: always_on
description: 1. Identify the change type: component, docs, docs example, analyzer, TS/style, asset pipeline, or metadata-only.
---

# AGENTS.md - AI Coding Agent Guide for MudBlazor

## Start Here

1. Identify the change type: component, docs, docs example, analyzer, TS/style, asset pipeline, or metadata-only.
2. Inspect nearby code and tests before editing.
3. Keep the diff scoped to the affected project or feature.
4. Use the smallest valid verification loop for the change type.
5. Run final whitespace formatting only for relevant changed source files.
6. In the final response, report changed areas, exact verification commands, and any skipped checks.

## Change Type Matrix

| Change type | Common locations | Verification | Notes |
| --- | --- | --- | --- |
| Component C#/Razor behavior | `src/MudBlazor`, `src/MudBlazor.UnitTests*` | Filtered `dotnet test` on `MudBlazor.UnitTests.csproj` | Use `/p:SkipBunCompile=true` unless assets are affected. |
| Component public API | Component, tests, docs | Unit tests plus relevant docs validation | XML docs and `[Category(...)]` are required. |
| Docs page/example | `src/MudBlazor.Docs*` | Relevant docs build or generated docs tests | Do not edit generated docs tests. |
| TS/style/assets | `TScripts`, styles, `wwwroot`, asset inputs | Normal scoped build | Do not use `/p:SkipBunCompile=true`. |
| Analyzer/code fix | `src/MudBlazor.Analyzers*` | Filtered analyzer tests | Keep diagnostics, fixes, and tests aligned. |
| Metadata/prose only | Root markdown, `.github` text | No `dotnet` verification | Do not run build/test/format for prose-only changes. |

## Scope and Workflow

### Keep changes focused
- Target specific projects only. Solution-wide commands are too slow unless explicitly requested.
- Keep diffs small and focused. Avoid repo-wide rewrites unless explicitly asked.
- Prefer targeted, non-breaking changes unless the task explicitly requires broader or breaking work.
- If broader follow-up improvements are identified, suggest them for a separate PR instead of expanding the current diff.
- Do not add new heavy dependencies or packages without approval.
- Do not make speculative large changes when the intent is unclear. Ask a clarifying question or propose a short plan instead.

### Default working rules
- Follow `src/.editorconfig`.
- Treat warnings as errors. Do not ignore analyzer warnings.
- Do not run solution-wide commands unless explicitly requested.
- Do not make `dotnet clean` part of the normal local loop. Use it only when incremental build state is clearly stale or corrupted.
- If no code, project, test, docs app, or asset-pipeline inputs changed, do not call `dotnet`. Changes limited to files such as `README.md`, changelog text, issue templates, or other repo metadata do not require restore, build, test, or format.
- Prefer a single scoped `dotnet build` or `dotnet test` command as the first verification step. Split build and test only when you will reuse the build outputs for multiple test runs.
- Do not build `src/MudBlazor/MudBlazor.csproj` immediately before testing `src/MudBlazor.UnitTests/MudBlazor.UnitTests.csproj`; the test project already builds `MudBlazor`, `MudBlazor.UnitTests.Shared`, and `MudBlazor.UnitTests.Viewer`.

## Before Editing

- Search for existing patterns before adding helpers, abstractions, or new APIs.
- For component behavior changes, identify the likely unit test file before editing.
- For public API changes, identify the docs page and examples that may need updates.
- For TS, style, or asset changes, check whether `entrypoint.js` or generated assets are affected.
- If the task is ambiguous and a wrong assumption could cause a broad change, ask a focused clarifying question.

## Repository Layout

- `src/` contains the product code and nearly all project work. Expect the main library, docs app, tests, analyzers, benchmarks, and related support projects to live here.
- `src/MudBlazor/` is the core component library. Most component, utility, styling, `TScripts`, and `wwwroot` changes land here.
- `src/MudBlazor.UnitTests*` contains test projects and test support code. Look here for component tests, shared test infrastructure, viewer-only helpers, and docs-related tests.
- `src/MudBlazor.Docs*` contains the documentation site, examples, and docs build support. Update docs here when component behavior or public API changes.
- `src/MudBlazor.Analyzers*` contains analyzer, code-fix, and analyzer-test projects.
- Repo-wide build configuration is centered in `src/`, especially `src/Directory.Build.*` and `src/.editorconfig`.
- Tooling and automation live primarily in `tools/`, `.config/`, and `.github/`.
- Treat `bin/`, `obj/`, `TestResults/`, generated files, and similar outputs as build artifacts unless the task explicitly targets them.

## Environment Requirements

- The required .NET SDK is defined in `global.json`; use that version to restore, build, and test this repository.
- The library targets `net8.0`, `net9.0`, and `net10.0`.
- Verify the active SDK with `dotnet --version`.

## Scoped Commands and Verification

### Project targets
- Components: `src/MudBlazor/MudBlazor.csproj` and `src/MudBlazor.UnitTests/MudBlazor.UnitTests.csproj`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MudBlazor/MudBlazor](https://github.com/MudBlazor/MudBlazor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
