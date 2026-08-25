---
trigger: always_on
description: Lightweight cross-platform RF circuit simulator (DC, S-parameters, harmonic balance,
---

# circuitRF

Lightweight cross-platform RF circuit simulator (DC, S-parameters, harmonic balance,
loadpull/sourcepull). **NOT a SPICE simulator.** See `docs/PRD.md` for scope, the five
hero circuits, and non-goals. This file is standing project memory — keep it current.

## Searching this repo

**Use `grep` (or ripgrep) directly whenever possible instead of spawning a search agent.** This
repo's structure is well-known and its files are plain text — a targeted `grep -n` finds a
symbol, class, or XAML control faster and far cheaper than delegating a "find X" task to an
agent. Reach for an agent only when the search genuinely needs multi-step reasoning across many
unrelated locations, not for straightforward lookups.

## Resolving Issues
If significant findings were found during bug fixes or changes, write to the relevant RESOLVED.md 
files never CLAUDE.md. This helps keep the CLAUDE.md files small.

## Stack
- .NET 10 (LTS), C# 14
- Avalonia 12 (UI), SkiaSharp (canvas rendering), CommunityToolkit.MVVM (MVVM)
- CSparse.NET (sparse complex LU for large MNA), NumFlat (dense linear algebra)
- `RfCore` (Touchstone I/O, network params, the `DataSet`/`DataCube` result types, interpolation,
  renormalization, plotting) is **a first-class project of this repository, exactly like `Core`,
  `Engine`, and `Ui`** — `src/RfCore/` with its tests at `tests/RfCore.Tests/`, both listed in
  `circuitrf.slnx`, referenced via ordinary `ProjectReference`.

  **It is NOT a subtree, and there is nothing left to "un-subtree" (2026-07-30).** It arrived via
  `git subtree add` on 2026-07-29 purely to preserve history
  (brief-housekeeping-tearoff-palette-repo.md §6 — splotRF, the other consumer of the standalone
  RfCore repo, was being retired). **"Being a subtree" is not a persistent state in git**: there is
  no `.gitmodules`, no config, no live link to anything. RfCore's 24 original commits are a
  permanent *second parent* of merge `0bd04db`, and `git blame` on any file under `src/RfCore/`
  still resolves to its original author and date. The only residue is a three-line
  `git-subtree-dir/-mainline/-split` trailer in that one old commit message, which is inert unless
  someone runs `git subtree pull` — **so don't.** Treat `src/RfCore/` as ordinary first-party code.

  *Known git wrinkle, not a history loss:* `git log --follow <path>` does not cross the merge (a
  documented `--follow`-vs-merges limitation). `git blame` does. To read the pre-merge history
  directly: `git log 0bd04db^2 -- src/Data/DataCube.cs` (the *old* path, on the pre-merge parent).

  **The architectural boundary is unchanged and does not depend on directory placement** — it is
  enforced by assembly-reference checks in `tests/Firewall.Tests`, which is why moving RfCore under
  `src/` cost nothing. RfCore still references no UI framework, and nothing in it may.

## Build / test / run
- Build:   `dotnet build`
- Test:    `dotnet test`
- Run CLI: `dotnet run --project src/Cli -- <args>`
  Verbs: `sparam`, `dc`, **`hb`**, `elab`. `hb` runs the netlist's harmonic-balance analysis —
  single- or multi-tone — and runs the whole sweep when a `parametric_sweep` wraps it (naming the
  inner HB is promoted to its wrapper, since running the inner alone silently drops the sweep axis).
  It evaluates the TestBench's `measure` lines exactly as the GUI does, so a `.cnl` that works
  headless works when opened. `--set var=expr` overrides a global before elaboration;
  `-o out.{mat,npy,txt}` exports.
- Package: one script per platform, all writing to `dist/` — `packaging/windows/build-msi.ps1`
  (.msi, x64/arm64/x86), `packaging/macos/build-dmg.sh` (.dmg, Apple Silicon; wraps the existing
  `src/Ui/bundleFor*MacOS.sh`), `packaging/linux/build-deb.sh` (.deb, x64/arm64, needs `fpm`).
  **Each must run ON its own platform** (WiX is Windows-only, `codesign`/`hdiutil` macOS-only, and
  the Windows PE icon is only embedded when the publish happens on Windows). Step-by-step
  instructions live in `BUILDING.md`, which `README.md` links to; keep the two in step.
  App icons (`.icns`/`.ico`/`.png`) are **build products** rasterised from the committed brand SVGs
  by `dotnet run --project tools/IconGen`, which every packaging script runs first — no icon binary
  is ever committed.

  **Two packaging rules exist because breaking either fails silently** (both held by
  `tests/Ui.Tests/PackagingScriptTests.cs`, both learned from a real Windows build, 2026-08-18):
  - **Every `.ps1` under `packaging/` must be pure ASCII.** Windows PowerShell 5.1 reads a BOM-less
    `.ps1` as cp1252, so a UTF-8 emoji or box-drawing char decodes to bytes 0x93/0x94 — the curly
    quotes `“ ”`, which PowerShell honours as string delimiters. Nothing errors: the parser swallows
    everything to the next quote-class byte, PRINTS it instead of running it, and continues. One `📦`
    turned the whole `dotnet publish` block into a string literal (verified against the AST, lines
    48-54), and the first visible symptom was a `Get-ChildItem` "cannot find path …\publish\win-x64"
    from a *later* step. A BOM also fixes it and is the wrong fix — invisible, and it does not
    survive an editor round-trip anyone would notice.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [potatobeanradio/circuitRF](https://github.com/potatobeanradio/circuitRF) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
