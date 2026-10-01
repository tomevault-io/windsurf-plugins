---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A .NET Standard 2.0 wrapper around the `ffmpeg` and `ffprobe` CLIs, published to NuGet as `FFMpegCore` plus three optional extension packages. It shells out to real binaries — nothing is linked natively — so every integration test needs `ffmpeg`/`ffprobe` on `PATH`.

## Commands

```bash
dotnet test FFMpegCore.sln                                              # all tests (needs ffmpeg on PATH)
dotnet test FFMpegCore.sln --filter "FullyQualifiedName=FFMpegCore.Test.VideoTest.Video_ToMP4"  # one test (= is exact; ~ is substring)
dotnet test FFMpegCore.sln --filter "FullyQualifiedName~ArgumentBuilderTest"     # one class
dotnet format FFMpegCore.sln --severity warn --verify-no-changes        # lint, as CI runs it (drop --verify-no-changes to fix)
dotnet pack FFMpegCore.sln -c Release                                   # packages land in nupkg/
```

`TreatWarningsAsErrors` is on solution-wide, which turns NuGet's vulnerability-audit warning (NU1900) into a restore failure when the audit can't reach nuget.org. If restore fails with NU1900, append `-p:NuGetAudit=false`.

`.github/workflows/ci.yml` is the only workflow and has three jobs. `ci` runs on PRs to `main` and on pushes to `main`: the test matrix across Windows/Ubuntu/macOS with ffmpeg 8.1 (the version string is set per OS because Linux/Windows pull from BtbN's builds and macOS from osxexperts.net); lint runs on Ubuntu only. On pushes to `main`, `version` evaluates every csproj with `IsPackable=true` and asks nuget.org's flat-container index whether its `PackageVersion` already exists; `release` then fans out over the ones that don't and, per package, packs, pushes and creates a GitHub release. Bumping `PackageVersion` on `main` is therefore the release trigger — there is no release branch, tag push or button, and a push where every version is already on nuget.org is a no-op. `release` is gated on the full test matrix and is idempotent (`--skip-duplicate`; re-run it on failure). Tags are `vX.Y.Z` for `FFMpegCore` and `<PackageId>/vX.Y.Z` for the extensions; only `FFMpegCore` releases are marked "latest". Release notes are GitHub's auto-generated notes (merged PRs) between the package's previous tag and the release commit; the same text is written to the nupkg via `PackageReleaseNotesFile` (see `Directory.Build.props`), so don't set `PackageReleaseNotes` in a csproj — it would override the generated notes. Publishing uses trusted publishing: `NuGet/login` exchanges the job's GitHub OIDC token for a short-lived API key, so there is no NuGet secret in the repo. The matching policy lives on nuget.org under the `rosenbjergsoftworks` account (owner `rosenbjerg`, repo `FFMpegCore`, workflow `ci.yml`, no environment) — renaming the workflow file or the repo requires updating that policy.

## Solution layout

| Project | Purpose |
|---|---|
| `FFMpegCore` | Core library (netstandard2.0). Depends only on `Instances` (process wrapper) and `System.Text.Json`. |
| `FFMpegCore.Extensions.SkiaSharp` / `.System.Drawing.Common` | In-memory `Snapshot` → bitmap, and `BitmapVideoFrameWrapper` for piping frames in. Separate packages because the core must not depend on an image library. |
| `FFMpegCore.Extensions.Downloader` | Downloads ffmpeg binaries from the ffbinaries API at runtime. |
| `FFMpegCore.Test` | MSTest, net8.0. |
| `FFMpegCore.Examples` | Console sample; not packed. |

`Directory.Build.props` sets the shared defaults (netstandard2.0, nullable, implicit usings, warnings-as-errors). Test and Examples override to net8.0; the test project disables nullable.

Each packable csproj carries its own `PackageVersion`; there is no central version file. Bumping it and merging to `main` is what releases that package (see CI above). Don't add `GeneratePackageOnBuild` — it makes `dotnet pack` skip the build and fail with NU5026 on a clean checkout.

## Core architecture

### The fluent argument pipeline

```
FFMpegArguments.From*Input(...)      // adds input(s); each is an IInputArgument
    .AddFileInput / .AddPipeInput    // more inputs
    .OutputToFile / .OutputToPipe    // adds the IOutputArgument, returns FFMpegArgumentProcessor
    .NotifyOnProgress / .CancellableThrough / .Configure
    .ProcessSynchronously() / .ProcessAsynchronously()
```

- `FFMpegArgumentsBase` holds a flat `List<IArgument>`. `FFMpegArguments.Text` joins every argument's `Text` in insertion order. Options passed via the `addArguments` lambda are appended **before** the input/output they belong to, so `-ss` lands before `-i` and `-c:v` lands before the output path. Order is significant to ffmpeg; don't reorder the list.
- Every ffmpeg flag is its own class in `FFMpegCore/FFMpeg/Arguments/` implementing `IArgument` (`Text` property), exposed through a `With*` method on `FFMpegArgumentOptions`. Adding a flag means: new `XxxArgument`, new `WithXxx` on `FFMpegArgumentOptions`, and an exact-string assertion in `ArgumentBuilderTest`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rosenbjerg/FFMpegCore](https://github.com/rosenbjerg/FFMpegCore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
