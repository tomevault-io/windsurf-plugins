---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Privacy — never commit real project data

This repo is public. **Never add real customer/project ST files, machine diagnostics, ROT
dumps, IP addresses, or AMS Net IDs.** Test fixtures in `samples/` must be **synthetic** —
write generic examples that exercise the construct, never copy files from the user's
TwinCAT projects or `Documents\TcXaeShell`. `samples/RealUnformatted/`, `docs/baseline-*.txt`,
and `docs/after-*.txt` are git-ignored for this reason.

**Rule — rename every identifier when turning real code into a test.** When you reproduce a
bug from a user's real code, use that code only locally (e.g. from the Host log) and, before
committing any test or fixture for it, **change all the code names**: variable names,
FB/POU/DUT/GVL names, method names, project names, file paths, and comments must be replaced
with generic placeholders (`a`, `b`, `fbSample`, `ST_Sample`, `MAIN`, …). The committed
fixture must reproduce the *construct* that broke, never the user's real naming — even when
the rest is synthetic. If you cannot rename it safely, do not commit it.

## Versioning (SemVer, dev/release)

`Directory.Build.props` `<VersionPrefix>` is the **single source of truth** for the version
— the only number to edit. Everything else derives from it: assembly versions, the CLI
`stfmt --version`, the Host doctor report, and the installer (`build-installer.ps1` reads
`VersionPrefix`; never hand-edit the version in `STFormatter-Setup.iss`).

- **Dev builds** (default): `dotnet build` produces a pre-release `X.Y.Z-dev+<gitShortSha>`,
  so every dev binary is traceable to a commit.
- **Release builds**: `dotnet build -p:PublicRelease=true` drops the suffix → clean
  `X.Y.Z`. Releases are git-tagged `vX.Y.Z`.
- **Bump rule**: feature → minor, fix → patch, breaking → major. Between releases the
  `VersionPrefix` already points at the *next* version with `-dev`.
- To cut a release, run the **`/release` skill** (`.claude/skills/release/`): it bumps the
  version, moves the CHANGELOG `[Unreleased]` block to the new version, builds, commits,
  tags `vX.Y.Z`, then sets the next `-dev` line. It never pushes — you push tags yourself.

## Read AGENTS.md First

[AGENTS.md](AGENTS.md) is the TcXaeShell integration survival guide. It documents, with evidence, why
VSPackage/MEF/AddIn approaches **do not work** in TcXaeShell and must never be retried, and how the
working external-process COM DTE approach functions (ROT monikers, clipboard live-edit, deployment
rules). Any work touching `STFormatter.Host`, `STFormatter.UI`, deployment, or the installer requires
reading it.

Non-negotiable rules from it:
- TcXaeShell integration is an **external process via COM DTE** only — never VSPackage, MEF, VSIX, or AddIn.
- Never deploy STBud files into Beckhoff's TcXaeShell `Extensions` tree; the Host lives in `C:\Program Files (x86)\STBud\`.
- Live edit uses DTE `Edit.SelectAll/Copy/Delete/Paste` + Win32 clipboard API (not `System.Windows.Forms.Clipboard`, not `TextSelection.Text`).
- Search the ROT for both `!TcXaeShell.DTE.*` and `!VisualStudio.DTE.*` monikers.

## Commands

```powershell
# Build everything
dotnet build TwinCAT.STFormatter.sln

# Run all tests (net8.0, xUnit)
dotnet test tests/STFormatter.Core.Tests

# Run a single test class / test
dotnet test tests/STFormatter.Core.Tests --filter "FullyQualifiedName~FormatterTests"
dotnet test tests/STFormatter.Core.Tests --filter "DisplayName~MethodName"

# Run the CLI
dotnet run --project src/STFormatter.CLI -- format <file> [--dry-run]
dotnet run --project src/STFormatter.CLI -- batch ./samples/RealTcFiles --twincat --dry-run

# Build + deploy the Host to C:\Program Files (x86)\STBud (self-elevates via UAC).
# deploy.ps1 stops the Host, copies, VERIFIES every file (timestamp+length; exits 1 and
# reports if anything is stale), then restarts the Host. deploy.bat is a thin wrapper.
dotnet build src/STFormatter.Host/STFormatter.Host.csproj -c Debug
deploy.bat            # net48 (default)
deploy.bat net462     # older machines
deploy.bat -NoPause   # automation (no interactive pause)

# Host log when debugging TcXaeShell integration (single log; DiffViewer logs here too,
# rotates at 5 MB to STBud_Host.log.1)
Get-Content "$env:TEMP\STBud_Host.log" -Tail 20

# Build the installer (Inno Setup)
installer\build-installer.ps1
```

## Architecture

STBud for TwinCAT is a toolbox for Beckhoff TwinCAT Structured Text; the core tool is an
IEC 61131-3 ST formatter. Four projects in one solution (assemblies keep the legacy
`STFormatter.*` prefix; the product name is STBud):

- **STFormatter.Core** (net8.0 / net48 / net462) — the formatting engine. Pure pipeline:
  `SourceText → Lexer → Parser → SyntaxTree → FormattingVisitor → FormattingWriter`.
  Hand-written lexer and recursive-descent parser produce an immutable syntax tree covering the
  full ST grammar plus TwinCAT extensions (`__TRY`, pragmas, access modifiers, actions).
  `FormattingEngine` exposes three entry points: `Format()` for full compilation units,
  `FormatBody()` for bare implementation bodies (wraps them in a temporary `PROGRAM`), and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [muratbilge/STBud-for-TwinCAT](https://github.com/muratbilge/STBud-for-TwinCAT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
