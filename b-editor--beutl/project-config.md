---
trigger: always_on
description: Common contract for any AI coding agent working in this repo (Claude Code, Codex, Cursor, etc.). Claude Code imports this file via `CLAUDE.md`.
---

# AGENTS.md — Beutl shared instructions (for all AI coding agents)

Common contract for any AI coding agent working in this repo (Claude Code, Codex, Cursor, etc.). Claude Code imports this file via `CLAUDE.md`.

## Language policy

- **Conversation & plans**: respond in the user's language (Japanese when the user writes in Japanese). Plans presented via plan mode / equivalents (e.g. Claude Code's ExitPlanMode) follow the conversation language.
- **Artifacts** — code, code comments, commit messages, PR titles/descriptions, Issue titles/descriptions, and project documentation under `docs/` — write in **English**, regardless of conversation language.

## Project overview

Beutl is a cross-platform video editing / compositing application built on Avalonia. It is written in .NET and C# / XAML, targeting both `net10.0` and `net10.0-windows` (dual-target).

- License: the main app is **MIT**; `Beutl.FFmpegWorker` alone is **GPL-3.0-or-later** (a separate process)
- UI: Avalonia (XAML + ViewModel)
- Tests: NUnit + Moq under `tests/` (per-area projects, e.g. `tests/Beutl.UnitTests/`, `tests/Beutl.Graphics3DTests/`, `tests/SourceGeneratorTest/`, `tests/Beutl.FFmpegIpc.Tests/`). `tests/Beutl.Graphics3DTests/` is a Vulkan-gated NUnit suite that self-skips when no Vulkan device is available — see `tests/CLAUDE.md`
- E2E / headless-UI tests: `tests/Beutl.E2ETests/` (library-level) and `tests/Beutl.HeadlessUITests/` (drives the real shell, the sole test referencing `src/Beutl`) on `Avalonia.Headless.NUnit`, with shared helpers in `tests/Beutl.Testing.Headless/`. They run on headless CI without xvfb or a GPU — see `tests/CLAUDE.md`
- Build: Nuke (`nukebuild/`) or `dotnet` directly

## Build / test / format

```bash
dotnet build Beutl.slnx                                            # build
dotnet test Beutl.slnx -f net10.0 --settings coverlet.runsettings  # test
dotnet format Beutl.slnx                                           # format
./build.sh <Target>                                                # Nuke (same as CI)
```

Claude Code skills are provided as `/beutl-build`, `/beutl-test`, `/beutl-format`, `/beutl-coverage`. They also fire on natural-language requests like "run the tests", "does this still build?", and they will **confirm scope** with AskUserQuestion (whole solution vs single project, verify vs apply, etc.) before executing. Pass arguments (e.g. `/beutl-test <FQN-substring>`) to skip the confirmation. Before opening a PR, `/beutl-pre-pr` runs the same checks locally that the CI review and the `beutl-reviewer` subagent will run.

## Module boundary map

| Project | Role |
|---|---|
| `Beutl.Engine` | Core rendering / scene / track (no project dependencies) |
| `Beutl.Engine.SourceGenerators` | Roslyn source generators |
| `Beutl.ProjectSystem` | Project / document persistence |
| `Beutl.Editor` | Non-UI editor logic — undo/redo, packaging, editing-pipeline services (no Avalonia) |
| `Beutl.Editor.Components`, `Beutl.Controls` | Avalonia UI layer (views / controls / ViewModels) |
| `Beutl.Extensibility` | Plugin abstractions |
| `Beutl.NodeGraph` | Node editor |
| `Beutl.FFmpegIpc` | **MIT** IPC layer (transport: Protocol / Transport / SharedMemory). Also hosts the IPC client providers under `Providers/`, which translate frame/sample messages into `Beutl.Media` / `Beutl.Extensibility` types; that adapter role is why this project deliberately takes `ProjectReference`s to `Beutl.Engine` + `Beutl.Extensibility` rather than staying dependency-free. |
| `Beutl.FFmpegWorker` | **GPL** separate process; reach it only via IPC |
| `Beutl.Api` | Server API client |

## Mandatory rules

1. **Do not cross the GPL/MIT boundary.** MIT projects must not take a compile-closure `ProjectReference` to `Beutl.FFmpegWorker`; the sole exception is `src/Beutl/Beutl.csproj`'s build-order-only reference (`ReferenceOutputAssembly="false"` + output-copy target). The `.claude/hooks/check-gpl-mit-boundary.sh` PreToolUse hook enforces this mechanically. Details: `docs/ai-workflow/gpl-mit-boundary.md`.
2. **XAML must use compiled bindings** — every new UserControl declares `x:CompileBindings="True"` together with `x:DataType="..."`. Details: `.claude/rules/xaml.md`.
3. **New logic ships with a NUnit test** — add tests under `tests/` in the matching test project (e.g. `tests/Beutl.UnitTests/` for general unit tests, `tests/SourceGeneratorTest/` for generator changes).
4. **Do not ask the AI to do the linter's job** — `.editorconfig` / `xamlstyler.json` / `dotnet format` own style.
5. **Do not change existing CI workflows (`.github/workflows/*`) without explicit approval.**
6. **Force-pushing to `main` / `master` is forbidden** — the hook denies the literal `git push (--force | -f | --force-with-lease) origin (main | master)` forms. Bypass routes (refspec forms like `HEAD:main`, variable expansion, etc.) are explicitly out of scope for the hook and are guarded by GitHub branch protection. Push to a feature branch instead.

## Design priorities (adopt better designs eagerly)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [b-editor/beutl](https://github.com/b-editor/beutl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
