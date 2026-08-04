---
trigger: always_on
description: Guidance for AI coding tools (Claude Code, GitHub Copilot, Cursor, Aider, Codex, etc.) working in this repo.
---

# AGENTS.md

Guidance for AI coding tools (Claude Code, GitHub Copilot, Cursor, Aider, Codex, etc.) working in this repo.

This is the **canonical brief**. GitHub Copilot reads this `AGENTS.md` natively; the `CLAUDE.md` tool-specific file points here.

## What this project is

**Fallout** — a build automation system for C#/.NET, hard-fork successor to [NUKE](https://github.com/nuke-build/nuke). The build is itself a C# console app (`build/_build.csproj`), so any change to the framework can be dogfooded by running `./build.ps1` (Windows) or `./build.sh`.

Originally NUKE by [matkoch](https://github.com/matkoch); under new maintenance as of 2026 and being renamed to Fallout. The codebase is mature, large, and has long-standing conventions — prefer matching existing patterns over introducing new ones.

**Rebrand status:** the structural rename has landed — namespaces (`Fallout.*`), package IDs, project filenames, and the global tool name (`dotnet fallout`) are all in place. Legacy `Nuke.*` lives on only as the consumer transition shims under `src/Shims/`. See [docs/rebrand-plan.md](docs/rebrand-plan.md) for the locked namespace mapping.

**Versioning & channels (calendar versioning, two-tier ladder — [ADR-0004](docs/adr/0004-calendar-versioning-and-dual-pace-channels.md), channel ladder superseded by [ADR-0008](docs/adr/0008-collapse-experimental-into-main.md)).** The project ships on **calendar versions `YYYY.MINOR.PATCH`** (mechanically valid semver; major = year). A maturity ladder feeds the production line — **GitHub Packages = test/preview; nuget.org = production**:
- **`main` = the integration trunk + sole `-preview` channel** — default branch; both deliberate improvements/bug fixes *and* faster AI-assisted work land here. Per-commit `-preview` prereleases (`2026.1.0-preview.<height>.g<commit>`) to **GitHub Packages only — never nuget.org**. Ordinary review. (The dedicated `experimental` `-alpha` lane was removed by ADR-0008 — it ran behind `main` and carried no unique work.)
- **`release/YYYY` = the production line** — **cut from `main` on demand at the first release of the year, not preemptively** ([ADR-0007](docs/adr/0007-cut-release-branch-on-demand.md)); until then `main` (`-preview`) is the most-stable line. Hardened deliberately, `-rc.N` → GA, non-breaking minors/patches only after the cut, rigorous review. Tags publish to nuget.org (opt-in) + GitHub Packages + GitHub Releases.
- **Breaking changes are batched to the yearly major cut** — they accumulate on `main` gated behind `[Experimental("FALLOUT0xx")]` (or a short-lived topic branch off `main` when they can't be gated) and ship as next year's `YYYY+1.0.0`. Mid-year `main`/production is strictly non-breaking; the production-cut review is the backstop. Version ladder: `-preview` < `-rc` < GA.
- **Legacy `support/v10`** (renamed from `release/v10`; + `hotfix/v10.x`) stays on semver `10.x`, security/critical fixes only; retired year lines become **`support/YYYY`**. **`release/v11` is retired and its branch removed** (nothing clean shipped; its work re-homed onto the 2026 line) — dead branches with no unique history are now deletable, tags are the durable release markers ([ADR-0007](docs/adr/0007-cut-release-branch-on-demand.md) §6).
- Opt-in unstable public APIs are marked `[Experimental("FALLOUT0xx")]` and can ride any channel; promoting to stable = removing the attribute.

**Active work** — rebrand completion + plugin-architecture internal foundation ([milestone #6](https://github.com/Fallout-build/Fallout/milestone/6)), now shipping on the `2026` line. **No public plugin SDK yet** — that's a later major ([milestone #7](https://github.com/Fallout-build/Fallout/milestone/7)). Internal middleware/listener interfaces stay `internal`; do not expose via `InternalsVisibleTo` to non-test assemblies. See [docs/roadmap.md](docs/roadmap.md) and the five open RFCs ([#97](https://github.com/Fallout-build/Fallout/issues/97)–[#101](https://github.com/Fallout-build/Fallout/issues/101)).

## Stack

- .NET SDK pinned in `global.json` (currently `10.0.100`, `rollForward: latestMinor`).
- Central package versions in `Directory.Packages.props` — never add a `Version=` to an individual `PackageReference`.
- xUnit + FluentAssertions + Verify.Xunit for tests.
- Solution file is `fallout.slnx` (new XML solution format, not `.sln`).
- Dependency updates: Handled by Dependabot (weekly grouped PRs).

## Common commands

```powershell
./build.ps1                          # default target = Pack
./build.ps1 Compile
./build.ps1 Test
./build.ps1 GenerateTools            # regenerate tool wrappers from JSON
./build.ps1 --help                   # list all targets and parameters

# Or via dotnet directly when iterating on a single project
dotnet build fallout.slnx
dotnet test tests/Fallout.Common.Tests/Fallout.Common.Tests.csproj
```

Do commit code generated by `GenerateTools` — the `.Generated.cs` files are checked in, and `VerifyGeneratedTools` fails CI if a `.json` spec edit isn't accompanied by regenerating and committing its wrapper.

To restructure an existing PR's commit history into focused commits, use the `/restructure-pr-commits` skill.

## Critical rules (read this every session)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Fallout-build/Fallout](https://github.com/Fallout-build/Fallout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
