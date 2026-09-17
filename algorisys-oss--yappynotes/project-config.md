---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## What this is

YappyNotes is a cross-platform desktop sticky notes app — C# / .NET 10, Avalonia
UI, SQLite — where each note is its own always-there window rather than a row in
a list. MVVM, with a repository behind the services.

Three files carry the project and they do not overlap:

- **[docs/plan.md](docs/plan.md)** — what we are building, why the architecture is
  shaped this way, and the milestones.
- **[LOOP.md](LOOP.md)** — how we build it. **This project is test-driven; read
  LOOP.md before writing code, every session.**
- **[README.md](README.md)** — how to run it.
- **[TODO.md](TODO.md)** — noticed since, not scheduled. Add to it rather than
  letting an idea live in a commit message; do not work from it without asking.

[docs/sticky-notes-architecture.pdf](docs/sticky-notes-architecture.pdf) is the
original whiteboard drawing and has no text layer, so it cannot be read by
grepping. `docs/sticky-notes-architecture.md` is the transcription; read that one.
Where it and `plan.md` disagree, `plan.md` is newer and wins.

The SDK is pinned to `10.0.302` in `global.json` and every project targets
`net10.0`.

**The project is at Milestone 5, done; Milestone 6 is half done — the tray icon is
in, rich text is not started.**
All eight MMF items hold — notes are their own draggable, resizable, pinnable,
recolourable windows, autosaved and restored; the manager lists, searches and
archives; there is a settings window, keyboard shortcuts, CI and packaging for
six runtime identifiers. On top of that, a note can carry a stream timer that
counts down or up, and the links in its text are offered beside it. The app
lives in the tray and outlives its windows, and an installed copy updates itself
from GitHub releases through Velopack. 394 green tests.

**What is left of Milestone 6 is rich text**, and it is a separate session's
work. `docs/plan.md` has the scope, the recommended approach and what was checked
already — read that section before starting, and in particular the argument for
keeping Markdown *in* `Content` rather than storing a rich-text blob: it is what
keeps search, export and the single-file goal intact. Seven other ideas are
parked there with their reasons, sync among them; it is rejected rather than
deferred.

**The app was called SmartNotes until it was renamed to YappyNotes.** Nothing in
the code carries the old name. `UserPaths.PreviousAppFolderNames` is the one
deliberate exception: it is how notes kept under the old name are adopted on the
first start afterwards. Append to that list if it is ever renamed again — an
entry removed is somebody's notes left behind.

`origin` is <https://github.com/algorisys-oss/yappynotes>, public.

## Commands

```bash
dotnet build yappynotes.sln
dotnet test yappynotes.sln
dotnet format yappynotes.sln

# One project's tests, one class, one test
dotnet test tests/YappyNotes.Core.Tests
dotnet test yappynotes.sln --filter "FullyQualifiedName~AutoSaveServiceTests"
dotnet test yappynotes.sln --filter "FullyQualifiedName~AutoSaveService_ClosingANoteMidDebounce_StillWrites"

# The fast tests, watched - keep this running while working
dotnet watch test --project tests/YappyNotes.Core.Tests

# Run the app
dotnet run --project src/YappyNotes.App/YappyNotes.App.csproj
scripts/dev-start.sh              # the same in Debug, so F12 developer tools exist
                                  # --watch to restart on a change
                                  # --sandbox for a throwaway database under artifacts/

# Install a release build for this machine into ~/Desktop/tools/yappynotes
scripts/deploy-local.sh           # or pass another tools folder

# The self-updating installer for one runtime, packed on its own OS
scripts/package-installer.sh linux-x64
```

Use `--sandbox` before touching the schema. Testing a migration against your own
week-old notes is how notes get lost.

### CI

`.github/workflows/ci.yml` builds, tests and format-checks on every push to
`main` and every pull request, then packages all six runtime identifiers. Ubuntu
only, because nothing in the suite needs a window. There are no skipped tests and
nothing that needs a database server — if CI is green and your machine is not,
the difference is yours.

### Packaging

`scripts/package.sh <rid>` builds a self-contained release for one of six runtime
identifiers and **prints the artifact path on stdout and nothing else** — build
logs go to stderr, because callers capture the path with `$(...)`. Keep it that
way, and add new packaging (a `.deb`, an `.app`) by calling it with
`--publish-only` rather than writing a second `dotnet publish`.

Releases are **not** single-file: Avalonia's native libraries want to be real
files on disk.

`scripts/version.sh` is the only reader of the version, and `Directory.Build.props`
the only place it is written: `VersionPrefix`, plus `VersionSuffix` for a
prerelease.

`scripts/package-installer.sh <rid>` packs Velopack's self-updating installer on
top of `package.sh --publish-only`, same stdout rule. `vpk` is pinned in
`dotnet-tools.json` and **packs only for the OS it runs on**, which is why

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [algorisys-oss/yappynotes](https://github.com/algorisys-oss/yappynotes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
