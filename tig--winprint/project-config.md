---
trigger: always_on
description: Cross-platform source/document printing engine. .NET 10. The Markdown work and the
---

# winprint — agent notes

Cross-platform source/document printing engine. .NET 10. The Markdown work and the
cross-platform CTE rendering refactor are described below because the "why" isn't
obvious from the code alone.

## Projects
- `src/WinPrint.Core` — engine. **Multi-targets `net10.0` and `net10.0-windows`.** The
  `WINDOWS` constant is defined for the `-windows` TFM only.
- `src/WinPrint.TUI` — Terminal.Gui front end and `wp` command.
- `src/WinPrint.Maui` — MAUI app (Windows/MacCatalyst; needs the `maui` workload — does
  **not** build on Linux).
- `tests/WinPrint.Core.UnitTests` — xUnit. Targets `net10.0-windows` but most tests run
  on Linux too (see below).
- Solution: `WinPrint.slnx`.

## Build & test
```bash
dotnet build src/WinPrint.Core/WinPrint.Core.csproj          # builds both TFMs
dotnet test  tests/WinPrint.Core.UnitTests/WinPrint.Core.UnitTests.csproj
dotnet test  ... --filter "FullyQualifiedName~CteRenderingTests"   # single class
```
CI (`.github/workflows/ci.yml`) runs on **windows-latest**, installs the `maui`
workload, builds `WinPrint.slnx`, then enforces a **style gate**. **Always run the gate
locally and get a clean tree before you push any C# change** — a failed style gate is the
single most common way agents waste a push/CI round-trip. There is exactly one command:

```bash
scripts/verify-style.sh                                            # whole solution
scripts/verify-style.sh src/WinPrint.Core/WinPrint.Core.csproj     # one project (Linux/Mac: MAUI won't build)
```

`scripts/verify-style.sh` **is** the CI gate (CI calls it via `--ci`), so a clean run there
means a clean run in CI. **Do not** hand-run bare `dotnet jb cleanupcode` / `dotnet format`:
they omit the exact profile/exclude flags CI uses (`--profile="WinPrintCleanup"
--exclude="**/*.xaml.cs"`), so they "pass" locally and still fail CI. If the script rewrites
files, commit those changes — that *is* the fix; re-run until `git diff` is empty. On Linux/Mac
(where `WinPrint.Maui` can't build) scope the script to the project(s) you touched; the full
solution gate over MAUI files is still verified by Windows CI. Code-style analyzers also enforce
**one top-level type per file** (WPA0001) and **no nested types** (WPA0002).

**CI mechanics & flakiness (don't mistake a flake for a real failure).** CI's `push` trigger is
scoped to `[main, develop, v2.1]` (not every branch), so a PR from a feature branch gets exactly
**one** run per push (the `pull_request` event) — this used to be two (a `push` run plus a
`pull_request` run on the same commit), which double-billed the `windows-latest`/`macos-26` jobs
for no extra coverage; a `concurrency` group also cancels a run outright when a newer commit
supersedes it. `maui-ui-tests` (Appium) and the FlaUI / TUI-golden suites are still **flaky** on
their own — a red run can just mean re-run it (`gh run rerun --failed <run-id>`) before assuming a
real regression.

## Remote (Claude Code on the web) environment
Fresh Linux containers have no toolchain. `.claude/hooks/session-start.sh` (registered
in `.claude/settings.json`) installs the .NET 10 SDK, `libgdiplus`, the local `jb` tool,
and warms NuGet restore. `global.json` pins .NET 10. The hook runs only when
`CLAUDE_CODE_REMOTE=true`.

## Release & distribution (read before cutting a release)
**Cutting a release.** Merge `develop` → `main`, create an **annotated** tag `vX.Y.Z` on the
merge commit, and `git push` the tag — that triggers `.github/workflows/release.yml`. There is
no release script; tags are manual. The pushed tag **also** triggers
`.github/workflows/back-merge.yml`, which opens a PR merging `main` back into `develop` — **merge
it** so `develop` doesn't silently drift behind `main` (it once fell ~43 commits behind). The tag
drives the brew/winget version; GitVersion drives the
Velopack `packVersion` — they coincide on a tagged commit. A pre-release label (`v…-rc.1`)
publishes as a GitHub *pre-release* (not "Latest"). A burned tag (release failed) can't be reused
— bump to the next patch. **A green release run can still mean "didn't publish":** if any
`Package <rid>` job fails, `Publish` / `winget` / `brew` are **skipped** and nothing ships, even
though the overall run may look done — always confirm a real GitHub release + tap update exist.

**Red release run? Triage before declaring the tag burned** (both bitten cutting v3.0.10):
- **A failed run is NOT a burned tag until a re-run also fails.** `gh run rerun --failed
  <run-id>` re-runs the failed `Package` legs and *resumes the skipped downstream jobs*
  (`Publish`/`winget`/`brew`) from the **same tag** — v3.0.10 shipped completely this way. Only
  a *deterministic* failure (fails identically on re-run) burns the tag and needs a patch bump.
- **Known release flake:** `Package win-x64` can die in the Velopack smoke test
  (`Test-WindowsVelopackShortcut.ps1`) with `Program 'wp.exe' failed to run:
  StandardOutputEncoding is only supported when standard output is redirected`. That is an
  intermittent **PowerShell** native-launch bug (oh-my-posh #1967, starship #6331) thrown in the
  *parent* before `wp` code runs — not a product regression; re-run it. If it ever fails this way

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tig/winprint](https://github.com/tig/winprint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
