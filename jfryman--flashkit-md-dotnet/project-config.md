---
trigger: always_on
description: Cross-platform .NET 10 port of krikzz's FlashKit MD programmer client
---

# CLAUDE.md

Cross-platform .NET 10 port of krikzz's FlashKit MD programmer client
(Sega Mega Drive / Genesis cart dumper/flasher). See docs/porting-plan.md
for the (completed, archived) staged porting plan and
docs/hardware-validation.md for real-hardware results.

## Docs map (keep the audiences separate)

- README.md — end users only: install from release artifacts, GUI/CLI
  usage. No build instructions there.
- DEVELOPING.md — contributors: build/test/publish, architecture,
  release process. Human-oriented prose.
- CLAUDE.md (this file) — agent working rules, condensed.
- docs/RELEASING.md — release signing/notarization secrets and behavior.

A user-visible change usually needs README.md updated; a workflow or
build change usually needs DEVELOPING.md. Update this file only for
rules agents must follow.

## Build and test

```
./eng/ci.sh        # restore + build (-warnaserror) + all tests; must stay under ~10s
./eng/publish.sh   # self-contained single-file binaries into artifacts/<rid>/
```

The .NET 10 SDK lives at `~/.dotnet` and is NOT on the default PATH.
Both scripts source `eng/ensure-dotnet.sh`, which puts it on PATH and
auto-installs the global.json-pinned SDK there if missing; for ad-hoc
commands use `export PATH="$HOME/.dotnet:$PATH"`.

Shared MSBuild settings (including `AnalysisLevel latest-recommended` +
`EnforceCodeStyleInBuild`) live in `Directory.Build.props`; NuGet
versions only in `Directory.Packages.props` (central package
management); the test projects' common xunit stack in
`tests/Directory.Build.props`. ci.sh builds `-warnaserror`, so a new
analyzer diagnostic fails CI: fix the code, or suppress narrowly with a
written justification (scoped `.editorconfig` section or
`[SuppressMessage]`) — the verbatim-ported `Device.cs`/`Cart.cs`, test
naming, and `flashkit_md` namespace exemptions are already in place.
Beware: MSBuild `-warnaserror` still writes outputs, so an immediate
rebuild can "succeed" incrementally — use `--no-incremental` when
verifying analyzer fixes.

## Changelog (permanent repo preference)

CHANGELOG.md follows the mitchellh/HashiCorp style: one `## X.Y.Z
(Month D, YYYY)` section per release with FEATURES / IMPROVEMENTS /
BUG FIXES headings and `component:`-prefixed entries (cli, gui, tui,
core, serial, release, docs). Every user-visible change adds an entry under `## Unreleased`
in the same commit as the change. To cut a release: rename Unreleased to
the version + date, commit, then tag `vX.Y.Z` — the release workflow
extracts that section for the GitHub release notes and fails the release
if the section is missing.

## Architecture (library-first)

- `src/FlashKit.Core/` — everything; front-ends only render.
  - `Device.cs` / `Cart.cs`: serial protocol + cart logic, ported VERBATIM
    from the original client (lowercase method names and all). Keep them
    diffable against the original source at
    https://github.com/krikzz/flashkit (flashkit-md/; also preserved in
    this repo's history, commit "Import pristine FlashKit MD v1.0.0.0
    source"); behavior changes belong in separate commits with tests,
    or in FlashKitSession.
  - `DeviceConnector` / `PortDiscovery`: per-OS port scanning with
    surfaced errors (Linux: ttyACM*/ttyUSB*; macOS: cu.usbmodem*/
    cu.usbserial*, never tty.* which block on open; Windows: COM*).
  - `FlashKitSession`: the front-end API — GetInfo/ReadRom/WriteRom/
    ReadRam/WriteRam/BakeSave. Synchronous, progress via
    `Action<OperationProgress>` (each phase starts with Done=0),
    `VerifyException` on read-back mismatch, no console/file I/O.
    GetInfo's CartInfo also carries SystemName (Mega Drive vs Sega 32X)
    and Region.
  - `IpsPatch` (Apply/Create) and `RomHash` (Crc32/Md5/Sha1): pure
    byte-array helpers, original code (not ported, so the verbatim rule
    above does not apply). Front-ends do the file I/O; every read/write
    path reports the hashes.
- `src/FlashKit.Presentation/` — shared presentation model for the
  interactive front-ends (GUI and TUI). `ProgrammerModel` owns ALL interactive
  behavior: held-session lifetime (the macOS tcdrain-wedge fix), the
  device gate, the poll state machine, auto-dump/auto-write (including the
  deliberately-not-identity-keyed insertion logic), the transaction log,
  and display strings. INotifyPropertyChanged + `IUserPrompts` for user
  decisions; must be called on the UI thread (see the class comment).
  New interactive behavior goes HERE, not in a front-end.
- `src/flashkit-md/` — CLI: arg parsing, file I/O, rendering over
  FlashKitSession directly (one-shot commands need no ProgrammerModel).
- `src/FlashKit.Gui/` — Avalonia adapter over ProgrammerModel: renders
  model properties into controls, implements IUserPrompts with
  StorageProvider pickers, drives the poll timer. Headless tests in
  `tests/FlashKit.Gui.Tests` drive the window against the fake device via
  the connector/file-picker seams in MainWindow.
- `src/flashkit-md-tui/` — Terminal.Gui adapter over ProgrammerModel,
  same panel roles and seams as the GUI. Terminal.Gui views work without
  Application.Init, so `tests/FlashKit.Tui.Tests` need no driver or main
  loop. Note: production threading relies on Terminal.Gui's main-loop

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jfryman/flashkit-md-dotnet](https://github.com/jfryman/flashkit-md-dotnet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
