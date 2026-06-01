---
trigger: always_on
description: > Your global `~/.claude/CLAUDE.md` rules apply (think first, check in before
---

# CLAUDE.md — WrapTune project context

> Your global `~/.claude/CLAUDE.md` rules apply (think first, check in before
> major changes, keep it simple, etc.). This file is project-specific context
> on top of those.

## What WrapTune is

WPF / .NET 8 desktop app for Windows. A friendly GUI around Microsoft's
`IntuneWinAppUtil.exe` (Win32 Content Prep Tool) — wraps installers into
`.intunewin` packages for Microsoft Intune deployment.

Repo: https://github.com/thefinder808/WrapTune (public, MIT)

## Quick orientation

```
WrapTune/
├── App.xaml / App.xaml.cs        WPF entry. App.xaml preloads Daylight theme
├── MainWindow.xaml / .cs         Main window + wrap logic + theme handling
├── AppSettings.cs                JSON-backed settings (LOCALAPPDATA — see below)
├── Themes/
│   ├── Daylight.xaml             Light palette ResourceDictionary
│   └── Midnight.xaml             Dark palette ResourceDictionary
├── WrapTune.csproj               .NET 8 WPF project (single-file self-contained)
├── WrapTune.ico                  Committed but generated — see Icon section
├── Build-Installer.ps1           dotnet publish → WiX → WrapTune.msi
├── Generate-Icon.ps1             "Bicolor Stack" icon — System.Drawing/GDI+
├── Installer/
│   ├── Installer.wixproj         WiX MSI project (NuGet pulls WiX SDK)
│   ├── Package.wxs               MSI package definition. Manufacturer="thefinder808"
│   ├── OptionsDlg.wxs            Custom installer dialog (desktop-shortcut toggle)
│   └── Bundled/                  IntuneWinAppUtil.exe goes here (gitignored)
└── .github/workflows/release.yml CI: build → sign → publish on v* tag push
```

## Local development (Windows)

```powershell
# One-time: drop IntuneWinAppUtil.exe into Installer/Bundled/
# (download from https://github.com/microsoft/Microsoft-Win32-Content-Prep-Tool)

# Run for dev:
dotnet run --project WrapTune.csproj

# Full build + MSI:
pwsh ./Build-Installer.ps1
```

The CI workflow handles releases — you do not build MSIs locally to release.

## Release flow

```powershell
git tag v1.2.3
git push origin v1.2.3
```

That's it. CI:

1. Regenerates `WrapTune.ico` via `Generate-Icon.ps1`
2. `dotnet publish` → self-contained single-file `WrapTune.exe`
3. Builds MSI via WiX
4. Signs MSI with Azure Artifact Signing (OIDC, no client secret)
5. Creates GitHub Release with `WrapTune.msi` attached

Takes ~3 minutes. Watch via `gh run watch <id>` or the Actions tab.

**Versioning**: SemVer. Bug fixes = patch (v1.1.1), new features = minor
(v1.2.0), breaking changes = major (v2.0.0).

The tag drives the MSI version end-to-end: CI's "Build MSI" step strips the
leading `v`, appends `.0` (matches .NET's 4-part AssemblyVersion/FileVersion
shape; MSI's `MajorUpgrade` comparison only looks at Major.Minor.Build), and
passes the result to `Build-Installer.ps1 -Version`. CI also enforces a
strict `vMAJOR.MINOR.PATCH` tag shape — pre-release labels like `v1.1.2-rc1`
are rejected up front instead of failing late inside WiX. The version then
flows to two places:

1. `Package.wxs` via `<DefineConstants>Version=$(Version)</DefineConstants>`
   in `Installer.wixproj` → `Version="$(var.Version)"`. Drives ARP entry
   version and `MajorUpgrade` comparisons (so upgrades cleanly replace old
   installs instead of side-by-side stacking).
2. `WrapTune.exe`'s AssemblyVersion / FileVersion via .NET SDK's default
   handling of `-p:Version=...` on `dotnet publish`. Works because
   `WrapTune.csproj` deliberately leaves these properties unset — if you
   ever add an explicit `<AssemblyVersion>` there, this implicit sync
   breaks silently. Keeps file metadata consistent with MSI metadata.

Local builds default to `0.0.0.0` — unambiguously not a release.

## Settings file location (important — easy to get wrong)

`%LOCALAPPDATA%\WrapTune\settings.json`

NOT next to the .exe. The app was migrated to this path in v1.0.2 so it can
run as a normal (non-admin) user even when installed per-machine in
Program Files. See `AppSettings.GetSettingsPath()`.

Schema:

```json
{
  "ExePath":      "C:\\...\\IntuneWinAppUtil.exe",
  "SourceFolder": "C:\\...",
  "OutputFolder": "C:\\...",
  "Theme":        "Daylight",
  "Overwrite":    true
}
```

`Save()` is wrapped in try/catch — silently swallows IO errors so the app
never crashes on exit. `Load()` is also robust (returns fresh defaults on
any parse/IO failure).

## Theme system (added in v1.1.0)

Two `ResourceDictionary`s in `Themes/`. Every control in `MainWindow.xaml`
binds via `{DynamicResource …}` to tokens like `Accent`, `Fg`, `WindowBg`,
`Success`, `Error`.

Switching happens in `MainWindow.xaml.cs:ApplyTheme(name)` — clears
`Application.Current.Resources.MergedDictionaries` and adds the new one.
All `DynamicResource` bindings refresh instantly.

Title bar chrome (light/dark) is keyed off the `DarkTitleBar` int resource
in each theme dictionary, read by `ApplyDarkTitleBar()` via DWM.

`App.xaml` preloads Daylight so there's no unstyled flash before the
constructor's `ApplyTheme(_theme)` call.

To add a new theme: copy `Daylight.xaml`, change the colors, save as
`Themes/Verdant.xaml` (or whatever), update `ApplyTheme()` to accept it,
update `BtnTheme_Click` to cycle through.

## Icon


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thefinder808/WrapTune](https://github.com/thefinder808/WrapTune) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
