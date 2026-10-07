---
trigger: always_on
description: This project is a C#/.NET 10 WinUI 3 desktop app that reads hardware sensors through
---

# FluentSensors Development Guidelines

This project is a C#/.NET 10 WinUI 3 desktop app that reads hardware sensors through
LibreHardwareMonitorLib and shows them in a native Windows 11 interface. It ships self-contained,
unpackaged on GitHub and as an MSIX in the Microsoft Store, and it requires administrator rights because the
sensor library talks to the PawnIO kernel driver, which is installed separately and never shipped with the app.

- Keep the work scoped to what was asked. Avoid opportunistic refactors, formatting churn, dependency
  bumps and drive-by renames.
- Read the surrounding code before adding an abstraction. Prefer the MVVM, service and singleton patterns
  that the file already uses.
- Preserve existing comments verbatim when you change the code around them.
- Everything is English: code, comments, commit messages and release notes. UI strings are written in
  English into `Strings/en-US/Resources.resw`, never hardcoded: XAML reads them through `x:Uid`, code
  through `AppStrings`. Technical terms (datasheet words like Cache, Rank, Logical Processors) go into
  `Terms.resw` instead and are read through `AppTerms`, because a setting can keep them in English. The
  other languages sit next to it, and a key missing there falls back to English.
- Always follow `.editorconfig`. Text files are LF; `.gitattributes` pins the checkout, and the CI
  `format` job fails on CRLF.
- Prefix types by where their data comes from: `Lhm` for types facing LibreHardwareMonitorLib, `Win` for
  Windows-native ones (WMI, Win32, DXGI).
- All UI updates from background threads go through `DispatcherQueue.TryEnqueue`. `LhmHardwareTreeService`
  already marshals its own mutations to the UI thread, so do not dispatch again in its consumers.
- Check `AppDistribution` before assuming how the app was installed. It is the single source of truth for
  the installer, portable and Microsoft Store channels, and for whether self-update is available at all.
- Treat sensor reading, taskbar embedding, window lifetime and settings persistence as high-risk areas.
- Never translate sensor names, hardware type names or CSV column headers. They come from
  LibreHardwareMonitorLib and are a contract with other tools and with the recorded files.
- Prefer targeted search over full file reads, and cap any command whose output could be large.

## Comment Style

A comment explains what a piece of code does and why it exists, not how the framework works underneath.
Weight scales with how non-obvious the reason is. A helper whose name already says everything needs no
comment at all.

The points below describe the style the code currently follows. They are recommendations, not entry
requirements: bring your own style if you prefer one, and expect these to shift over time.

- XML documentation comments are not used anywhere in this codebase. No `/// <summary>`, `/// <param>`,
  `/// <returns>` or `/// <inheritdoc/>`; plain `//` single-line comments throughout.
- Section headers inside a class are written as `// === section name ===`, lowercase, three equals signs,
  with a blank line above and below, rather than as multi-line banner dividers.
- Related fields are grouped under a `// topic name` or `// --- topic name ---` subheader, with short
  trailing comments on the individual lines.
- A comment that only restates the line below it, or that defends an option nobody took, is usually
  better left out.
- Comments are kept short: one or two lines for what the code does and the one thing that is not
  obvious, the unit once on a group line, a trailing comment in a few words, and no history of how the
  code got to where it is.

Four tags mark code that is not ordinary. Use each only for what it names:

- `// --- workaround: short name ---` for a bug in the platform or a third-party library, never for one
  of ours. State the problem, link the issue or the confirmed repro, then state the fix.
- `// --- memory leak: short name ---` for the WinUI 3 retained-instance pattern, where a window is
  hidden and reused because a real close never releases it.
- `// --- revisit: short name ---` for a deliberate temporary shape that has a named trigger for
  changing it.
- `// KNOWN UNRELIABLE:` for an external data source (WMI, firmware, driver) whose values cannot be
  trusted, paired with `(unreliable)` on the matching UI label.

XAML comments put two spaces inside the markers (`<!--  note  -->`). When the text does not fit on one
line it becomes a block with the markers on their own lines, rather than several stacked one-liners. A
double hyphen inside a comment is an XML parse error, so anomaly tags are written plainly there:
`<!--  workaround: short name  -->`.

## Project Structure

```text
FluentSensors/
├── Common/       cross-cutting helpers: Csv, Localization, Markdown, Sensors, UI
├── Controls/     reusable controls: InfoPopup, SensorGraph, SensorRow, SensorTile, Threshold,
│                 TimeRange
├── Core/         infrastructure: Lhm, Startup, StaticInfo, Taskbar, Update
├── Diagnostics/
├── Features/     one folder per screen or feature, view plus view model and any window it owns:
│                 AppStatus, CsvLogging, Performance, Sensors, Settings, Start, TaskbarWidget,
│                 Update, Widget

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cechout/fluent-sensors](https://github.com/cechout/fluent-sensors) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
