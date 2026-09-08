---
trigger: always_on
description: Gamepad-first file browser for Xbox (UWP), inspired by yazi's Miller-column UX (Parent |
---

# AGENTS.md — X-Files

## Project
Gamepad-first file browser for Xbox (UWP), inspired by yazi's Miller-column UX (Parent |
Current | Preview, live preview), but implemented natively in C#/XAML — no code/core reuse
from yazi (Rust, terminal-based, incompatible tech stack).

Sibling project: `../dosbox-pure-uwp` — some infra patterns (P/Invoke directory scanning,
gamepad input abstraction, manifest capabilities) are intentionally reused as documented
patterns, not shared code (different language, no shared dependency).

## Language
All documentation, code, comments, and commit messages MUST be in English.
(User may converse in Portuguese or English; agent responds accordingly.)

## Status
**Released** — latest release `v1.2.0.{build}` (see `docs/RELEASE.md`). Feature-complete
for the planned MVP plus post-MVP features (audio player, 31 visualizers, text editor,
QR file sharing, batch operations, ROM preview, PDF viewer, settings, log viewer).
Docs in `docs/` reflect the shipped state. See `docs/ROADMAP.md` for the remaining backlog.
Unit tests live in `tests/` (MSTest, linked-source, net8.0 — run on desktop, not UWP).

## Critical Rules
- **NEVER commit or push** without explicit user request. Stage changes only. Wait for
  "commit", "push", "faz o commit", etc.
- **This project builds on Windows with VS2026 (MSBuild found at
  `C:\Program Files\Microsoft Visual Studio\18\Community\MSBuild\Current\Bin\MSBuild.exe`).**
  VS2022 run/debug broke and was replaced. Always run build verification after
  structural changes: `& "C:\Program Files\Microsoft Visual Studio\18\Community\MSBuild\Current\Bin\MSBuild.exe" "XFiles.sln" /p:Configuration=Debug /p:Platform=x64 /t:Build /v:minimal`.
  Do NOT use the VS2022 MSBuild — a machine-level `VisualStudioVersion=18.0` env var
  makes it look for the v18.0 WindowsXaml targets that only exist under VS2026 (MSB4226).
- **x64 target primarily** (Xbox Series). Confirm ARM64 needs before adding that platform.
- **`broadFileSystemAccess` + `runFullTrust`** capabilities are required in the manifest
  for any filesystem code outside the app's sandboxed folders (see `docs/FILEBROWSER.md`
  and `docs/DEPLOY-XBOX.md`). Do not "simplify" file access to `StorageFolder` APIs without
  checking this — it breaks browsing external drives on Xbox.
- **No XAML controls with default Fluent Design chrome** — every interactive control must
  use a custom `ControlTemplate`/`Style` from `Theming/BladeTheme.xaml` (see
  `docs/UI-THEMING.md`, ADR-002 in `docs/DECISIONS.md`). Gamepad focus
  (`XYFocusUp/Down/Left/Right`, `IsFocusEngaged`) must still work — that's the whole reason
  XAML was chosen over Win2D. Audio visualizers are the one exception (Win2D — ADR-009).
- **Read `docs/DECISIONS.md` before proposing architecture changes** — several trade-offs
  (XAML vs Win2D, SharpCompress vs native 7z libs, no network browsing in MVP, SQLite
  metadata cache) were already debated and decided; don't re-litigate without new
  information.
- **Log everything. Every operation, every action, every exception must be logged.** Use
  the central `Log` static class (Serilog). Levels: `Log.Info()` for significant actions,
  `Log.Dbg()` for internal state, `Log.Verb()` for high-volume events (input, per-tick),
  `Log.Warn()` for recoverable failures, `Log.Err()` for exceptions. Prefix messages with
  class/method name (e.g. `Log.Dbg("FileOperations.Copy: ...")`). High-frequency hot paths
  use `#if` flags (e.g. `GAMEPAD_POLL_DEBUG`) — compiled out by default, enabled in csproj
  for debugging. Never swallow exceptions — always log them. See `docs/LOGGING.md`.
- **Release flow is automated via GitHub Actions.** To release: update `RELEASE-NOTES.md`,
  commit, tag `v{major}.{minor}.{patch}.{build}` (4-part), push tag. Workflow builds +
  packages + creates release with zip. See `docs/RELEASE.md`. Do NOT manually create
  releases via `gh release create` — the workflow handles everything.

## Architecture at a Glance
See `docs/ARCHITECTURE.md` for the full picture. Layers (top → bottom):
XAML Views (`Controls/*`) → Input (`InputRouter`, `GamepadInputService`) →
Navigation (`INavigable`, `ColumnNavigator`) → FileSystem (`DirectoryScanner`,
`ArchiveBrowser`, `FileOperations`, `FilePreviewService`) → Services (media, metadata,
PDF, QR share). Cross-cutting: `Metadata/*` (MusicBrainz/Deezer + SQLite cache),
`Audio/AudioLevelService` (AudioGraph + FFT), `Visualizers/*` (Win2D), `Log.cs`,
`Theming/BladeTheme.xaml`.

## Key Docs

| Doc | Purpose |
|---|---|
| `docs/SPEC.md` | Functional spec, MVP scope, done criteria |
| `docs/ARCHITECTURE.md` | Layered architecture, data flow, column model |
| `docs/GAMEPAD.md` | Button mapping, `INavigable` contract, `InputRouter` |
| `docs/FILEBROWSER.md` | `FileEntry` model, `DirectoryScanner` (P/Invoke), sorting |
| `docs/ARCHIVES.md` | zip/7z/rar via SharpCompress, archive-as-virtual-folder |
| `docs/UI-THEMING.md` | ControlTemplate conventions, BladeTheme |
| `docs/AUDIO-VISUALIZATION.md` | VU meter architecture, AudioGraph, FFT |
| `docs/AUDIO-VISUALIZERS.md` | 31 Win2D visualizers, registry, shader pipeline |
| `docs/FILETYPE-ICONS.md` | File-type icon mapping (Papirus, `-24.png` scheme) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marcelofrau/x-files-uwp](https://github.com/marcelofrau/x-files-uwp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
