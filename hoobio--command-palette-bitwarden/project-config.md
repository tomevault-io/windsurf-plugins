---
trigger: always_on
description: Guidance for AI coding agents (Copilot, Claude Code, etc.) working in this repository.
---

# Agent Instructions

Guidance for AI coding agents (Copilot, Claude Code, etc.) working in this repository.

## Local Build & Deploy

This is an out-of-process Command Palette extension (a packaged WinRT COM server). To see a code change live you must rebuild, re-register the package, and restart PowerToys.

VS Code task **Build, Kill & Deploy (Debug x64)** (`.vscode/tasks.json`) runs the full cycle: kill PowerToys, build, register the loose package, restart PowerToys. The equivalent commands:

```powershell
# 1. Stop PowerToys, Command Palette, and the extension host so files unlock
@('PowerToys','Microsoft.CmdPal.UI','HoobiBitwardenCommandPaletteExtension') +
  ((Get-Process | Where-Object ProcessName -like 'Microsoft.CmdPal.Ext.*').ProcessName | Select-Object -Unique) |
  Select-Object -Unique | ForEach-Object { Get-Process $_ -ErrorAction SilentlyContinue | Stop-Process -Force -ErrorAction SilentlyContinue }

# 2. Build (Debug stamps the side-by-side "Dev" channel identity)
dotnet build .\HoobiBitwardenCommandPaletteExtension\HoobiBitwardenCommandPaletteExtension.csproj -c Debug /p:Platform=x64

# 3. Register the loose layout, then restart PowerToys
$out = '.\HoobiBitwardenCommandPaletteExtension\bin\x64\Debug\net10.0-windows10.0.26100.0\win-x64'
Copy-Item "$out\..\..\..\..\..\Assets\*" "$out\Assets" -Recurse -Force
Add-AppxPackage -Register -Path "$out\AppxManifest.xml" -ForceUpdateFromAnyVersion
Start-Process 'shell:AppsFolder\Microsoft.PowerToysWin32'
```

Debug builds install as a separate **Dev** channel (`Hoobi.BitwardenCommandPaletteExtension.Dev`, display name suffixed `(Dev)`) so they sit side by side with a production install. Test against the **(Dev)** entry in Command Palette.

### Working in a git worktree (important)

`Add-AppxPackage -Register` deduplicates on package **identity + version**. The Dev identity and version (`1.10.0.0`) are the same across every checkout, so if a Dev package is already registered (e.g. pointing at the main checkout's `bin`), registering again from a worktree's `bin` is silently skipped (`Deployed.` prints, but `InstallLocation` does not change). You then run old code and think the change "did nothing".

When deploying from a worktree, **always remove-then-register and then verify** - registering in place is silently skipped:

```powershell
# 1. Remove any existing Dev registration (wherever it points), then register from THIS worktree
Get-AppxPackage -Name 'Hoobi.BitwardenCommandPaletteExtension.Dev' | Remove-AppxPackage -ErrorAction SilentlyContinue
$out = "<worktree>\HoobiBitwardenCommandPaletteExtension\bin\x64\Debug\net10.0-windows10.0.26100.0\win-x64"
Add-AppxPackage -Register -Path "$out\AppxManifest.xml" -ForceUpdateFromAnyVersion

# 2. VERIFY (do not skip): identity must be .Dev and InstallLocation must be your worktree bin
$p = Get-AppxPackage -Name 'Hoobi.BitwardenCommandPaletteExtension.Dev'
"$($p.Name) -> $($p.InstallLocation)"   # Name must end in .Dev; path must be your worktree
```

If `Name` has no `.Dev` suffix, or `InstallLocation` is not your worktree, the deploy is wrong - do not stop there.

### Release-build trap (this is the one that bites)

The per-channel identity is stamped into `Package.appxmanifest` at build time, and the stamp is **incremental**. If you run a `-c Release` build first (e.g. for lint/test parity) and then a `-c Debug` build in the same checkout, the Debug build can skip regenerating the output `AppxManifest.xml`, leaving it stamped with the **Release** identity. Registering that loose layout installs the *production* channel from your Debug bin - exactly the "I deployed but nothing changed / wrong channel" symptom.

Guard against it: when deploying from a worktree, do a clean Debug build with the channel pinned explicitly, and check the output manifest before registering:

```powershell
Remove-Item -Recurse -Force "<worktree>\HoobiBitwardenCommandPaletteExtension\bin\x64\Debug" -ErrorAction SilentlyContinue
dotnet build .\HoobiBitwardenCommandPaletteExtension\HoobiBitwardenCommandPaletteExtension.csproj -c Debug /p:Platform=x64 /p:PackageChannel=Dev
Select-String -Path "$out\AppxManifest.xml" -Pattern '<Identity Name'   # must show ...Extension.Dev
```

Stamping note: a Debug build rewrites the tracked `Package.appxmanifest` to the Dev identity as a side effect. **Don't commit that change** - `git checkout -- HoobiBitwardenCommandPaletteExtension/Package.appxmanifest` before committing.

## Git Commit Message Format

All commit messages MUST follow [Conventional Commits](https://www.conventionalcommits.org/) for release-please compatibility. Commits and PRs in this repo do NOT require an Azure DevOps work item number (this repo isn't associated with an ADO project).

### Required Format

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### Commit Types

Use these standard types (release-please compatible):

- **feat**: A new feature (triggers MINOR version bump)
- **fix**: A bug fix (triggers PATCH version bump)
- **chore**: Maintenance tasks, dependency updates, tooling changes (no version bump)
- **docs**: Documentation only changes (no version bump)
- **style**: Code style changes (formatting, no functional changes, no version bump)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hoobio/command-palette-bitwarden](https://github.com/hoobio/command-palette-bitwarden) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
