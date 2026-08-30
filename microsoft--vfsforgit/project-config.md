---
trigger: always_on
description: This file is for AI coding assistants (GitHub Copilot, Claude Code, Cursor, OpenAI
---

# Agent instructions for VFS for Git

This file is for AI coding assistants (GitHub Copilot, Claude Code, Cursor, OpenAI
Codex, Gemini CLI, Aider, etc.). It captures project-specific knowledge that isn't
obvious from a fresh `git clone` and which routinely trips up agents. See
[CONTRIBUTING.md](CONTRIBUTING.md) for coding standards (StyleCop rules, error
handling, exception logging) — do not duplicate those here.

## Repository layout

The build scripts (`scripts\Build.bat` and friends) put build outputs **one
level up** from the git working tree. The Readme documents the recommended
clone-into-`src\` convention, which matches the layout that `gvfs clone`
creates for end users:

```powershell
# Recommended (matches Readme.md and CI):
git clone https://github.com/microsoft/VFSForGit C:\Repos\VFSForGit\src
```

```
C:\Repos\VFSForGit\
├── src\                    ← git working tree (.git, GVFS.sln, all source)
├── out\                    ← build output (gitignored — it's outside the working tree)
└── packages\               ← NuGet cache
```

Cloning without the `src\` suffix also works — outputs just land one level
above wherever you cloned. The rest of this document and the project's own
scripts (CI, Readme) use the `src\` form, so commands assume it. **The
parent directory must be writable** (cloning to `C:\` itself won't work
because the build can't create `C:\out`).

All commands below run from the **enlistment root** (the parent of `src\`):

```powershell
cd C:\Repos\VFSForGit         # NOT C:\Repos\VFSForGit\src
```

This keeps `src\...` and `out\...` paths symmetric and matches what
`Build.bat` does internally.

## Build paths

`scripts\Build.bat` does a full installer build with NativeAOT publish plus
Inno Setup. That's ~5 minutes minimum, and AOT has no incremental support
— ilc relinks every executable on every `dotnet publish`. **Do not use
`Build.bat` for dev-loop iteration.** Pick the right path for what you're
doing.

### Path A — Unit-test inner loop (~10–15 s incremental)

For C# changes verified by unit tests only.

```powershell
dotnet build src\GVFS\GVFS.UnitTests\GVFS.UnitTests.csproj -c Debug
& "out\GVFS.UnitTests\bin\Debug\net10.0-windows10.0.17763.0\win-x64\GVFS.UnitTests.exe" --test Fully.Qualified.Class.Or.Method
```

Skips `dotnet publish`, AOT, native C++ projects, payload assembly, installer.

### Path B — Functional-test inner loop (~30–60 s incremental)

For changes that need the GVFS payload (`gvfs.exe`, hooks, service) but not
an installer. `PublishAot=false` skips ilc (~3–4 min saved);
`SkipCreateInstaller=true` skips Inno Setup (~95 s saved).
`GVFS.Payload` cascades to its dependencies (GVFS, GVFS.Mount, GVFS.Hooks,
GVFS.Service) via `ProjectReference`.

> **Prerequisite: the native C++ projects must already be built.** They are
> `.vcxproj` (see [Native C++ projects](#native-c-projects-need-msbuild-not-dotnet-build)
> below) and `dotnet publish` will not build them for you. If you have not
> already done a `Build.bat` once in this enlistment, build the native
> projects via VS MSBuild first (or run `Build.bat` once to populate `out\`,
> then iterate with the commands below). After that they are incremental and
> only rebuild when their own sources change.

```powershell
dotnet publish src\GVFS\GVFS.FunctionalTests\GVFS.FunctionalTests.csproj `
    -c Debug /p:PublishAot=false
dotnet publish src\GVFS\GVFS.Payload\GVFS.Payload.csproj `
    -c Debug /p:PublishAot=false /p:SkipCreateInstaller=true

src\scripts\RunFunctionalTests-Dev.ps1 Debug --test=GVFS.FunctionalTests.Tests.<Namespace>.<Class>.<Method>
```

`layout.bat` (invoked by GVFS.Payload) `xcopy`s from each project's `publish\`
or native-output directory — the C# projects do not require AOT, so
`PublishAot=false` produces a fully functional test payload. The native
hook binaries are copied straight from the vcxproj output.

`RunFunctionalTests-Dev.ps1` runs functional tests against the build output
without requiring admin or a system-wide install. It launches the test
service as a console process. Each invocation gets a unique service name
and data dir, so concurrent runs from different worktrees don't collide.

### Path C — Installer build (~5 min — only when you need an installer)

For producing an installable package (testing install/upgrade flows, or
shipping a build). This is the only correct use of `Build.bat`.

```powershell
# Build.bat takes (configuration, version, verbosity). The 0. prefix on the
# version tells GVFS to treat this as a development build and skip the
# server-side version check.
$v = & { $n = [DateTime]::Now; "0.2.$($n.ToString('yy'))$($n.DayOfYear.ToString('D3')).$([int]($n.TimeOfDay.TotalSeconds / 2))" }
src\scripts\Build.bat Debug $v minimal
```

Installer output: `out\GVFS.Installers\bin\Debug\win-x64\SetupGVFS.<version>.exe`.

## Native C++ projects (need MSBuild, not `dotnet build`)

These five projects are `.vcxproj` and require Visual Studio MSBuild with
the C++ workload. `Build.bat` invokes MSBuild for them automatically; if
you need to rebuild them outside `Build.bat`, use `msbuild`, not
`dotnet build`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/VFSForGit](https://github.com/microsoft/VFSForGit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
