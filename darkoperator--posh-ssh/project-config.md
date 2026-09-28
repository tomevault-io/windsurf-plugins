---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Posh-SSH is a PowerShell module providing SSH, SFTP, and SCP functionality for Windows PowerShell 5.1 and PowerShell 7.x. It wraps the SSH.NET library to enable SSH automation against Linux/Unix servers from Windows, Linux, and macOS hosts.

Current version: **4.0.0** (see `Posh-SSH/Posh-SSH.psd1`). The v4.0 line is built against vanilla **SSH.NET 2025.1.0** from NuGet — the legacy Cisco-patched `Renci.SshNetDev.dll` HintPath and the .NET Framework 4.7.2 project have both been removed.

## Architecture

### Codebase Structure

1. **C# Binary Module** (`Source/PoshSSH/PoshSSH.Core/`)
   - Single SDK-style csproj, target framework `netstandard2.0` (cross-platform)
   - Compiles to `PoshSSH.dll`
   - Auto-includes all `.cs` files under the project directory — no `<Compile Link>` indirection
   - Dependencies are pulled via `PackageReference` only: `SSH.NET 2025.1.0`, `PowerShellStandard.Library 5.1.1`
   - Subdirectories:
     - `Stores/` — Trusted host store implementations: `ITrustedHostStore`, `JsonTrustedHostStore`, `MemoryTrustedHostStore`, `OpenSSHTrustedHostStore`
     - `TrustedHost/` — Trusted host cmdlets rewritten in C# (`AddSshTrustedHost`, `GetSshTrustedHost`, `NewSshTrustedHost`, `RemoveSshTrustedHost`, `SetSshTrustedHost`)
   - Base class `NewSessionBase.cs` provides shared connection scaffolding (`ComputerName`, `Credential`, `Passphrase`, `.BeginProcessing`, `.CreateConnection`, `.ProcessRecord`) for the session-creating cmdlets
   - Connection info assembly lives in `ConnectionInfoGenerator.cs`
   - Progress reporting helper in `OperationProgressHelper.cs`

2. **PowerShell Script Module** (`Posh-SSH/Posh-SSH.psm1`)
   - ~35 PowerShell functions wrapping the binary cmdlets and the SSH.NET API directly
   - Manages global session state via `$Global:SshSessions` and `$Global:SFTPSessions` ArrayLists
   - Pre-loads `Assembly/BouncyCastle.Cryptography.dll` and `Assembly/Renci.SshNet.dll` via `Add-Type` at the top of the file — this is what avoids assembly-resolution conflicts when something else in-process has a different Renci version loaded (e.g., Windows Server 2019 Storage Server's built-in copy)

### Cmdlets Exposed (14 total, declared in `Posh-SSH.psd1` `CmdletsToExport`)

Session and file-transfer:
`New-SSHSession`, `New-SFTPSession`, `Get-SCPItem`, `Set-SCPItem`, `Get-SFTPItem`, `Set-SFTPItem`, `Get-SSHHostKey`

Trusted host store backends:
`New-SSHMemoryTrustedHostStore`, `Get-SSHJsonTrustedHostStore`, `Get-SSHOpenSSHTrustedHostStore`

Trusted host management (C# rewrites from v3.x PowerShell originals):
`New-SSHTrustedHost`, `Add-SSHTrustedHost`, `Get-SSHTrustedHost`, `Remove-SSHTrustedHost`

### Module Manifest

`Posh-SSH/Posh-SSH.psd1`:
- `RequiredAssemblies = @('Assembly\Renci.SshNet.dll', 'Assembly\BouncyCastle.Cryptography.dll')`
- `NestedModules = @('PoshSSH.dll','Posh-SSH.psm1')`
- `Prerelease` is unset for stable releases; set it (e.g. `'beta1'`) to publish a prerelease such as `4.0.0-beta1`

### Session Management

- Each session type (SSH/SFTP) has a dedicated ArrayList
- Sessions carry `SessionId` (integer index), `Host`, and the underlying SSH.NET connection objects
- Functions use parameter sets to accept either `SessionId` (index) or session objects directly

## Development Commands

### Building

```powershell
.\Build-Module.ps1                        # Release build + package + SHA256
.\Build-Module.ps1 -Configuration Debug   # Debug build
.\Build-Module.ps1 -SkipBuild             # Repackage existing DLL only
.\Build-Module.ps1 -OutputPath C:\Releases # Custom zip location
```

What the script does:
1. `dotnet build -c $Configuration` on `Source/PoshSSH/PoshSSH.Core/PoshSSH.Core.csproj`
2. Copies the netstandard2.0 build output to `Posh-SSH/PoshSSH.dll`
3. Parses the manifest with `Test-ModuleManifest`
4. Spawns a fresh `pwsh`/`powershell` and verifies every cmdlet declared in `CmdletsToExport` actually loads (and warns if extras are exported)
5. Validates every `RequiredAssemblies` and `FileList` entry exists on disk
6. Packages `Posh-SSH/` into `Posh-SSH-{version}.zip` and reports the SHA256

The cmdlet expectations are derived from the manifest itself — no hardcoded list — so cmdlet renames in `Posh-SSH.psd1` flow through automatically.

### Documentation Generation

```powershell
.\Invoke-DocumentationBuild.ps1
```

Requires the `platyPS` module. Converts `docs/*.md` to MAML help in `Posh-SSH\en-US\` and refreshes the markdown from the loaded module.

### Testing

The `tests/` directory contains:

| File | Purpose |
|---|---|
| `Get-SSHSession.Tests.ps1`, `Remove-SSHSession.Tests.ps1` | Pester unit tests for session-management functions |
| `Posh-SSH.Integration.Tests.ps1` | Full integration suite against a live SSH server (password and key auth, multiple key formats) |
| `Run-IntegrationTests.ps1` | Interactive launcher for the integration suite |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [darkoperator/Posh-SSH](https://github.com/darkoperator/Posh-SSH) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
