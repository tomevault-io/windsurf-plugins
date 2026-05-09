---
trigger: always_on
description: This guide provides AI coding assistants with essential commands, patterns, and conventions for working in the WinMole codebase.
---

# AGENTS.md - Development Guide for WinMole

This guide provides AI coding assistants with essential commands, patterns, and conventions for working in the WinMole codebase.

**Quick reference**: Build/test commands, Safety rules, Architecture map, Code style

---

## Safety Checklist

Before any operation:

- Use `Remove-SafeItem` helpers (never raw `Remove-Item -Recurse -Force`)
- Check protection: `Test-ProtectedPath`, `Test-WhitelistedPath`
- Test first: `-WhatIf` parameter or `$env:WINMOLE_DRY_RUN = 1`
- Validate syntax: PowerShell parser
- Run tests: `Invoke-Pester -Path .\tests\`

## NEVER Do These

- Run `Remove-Item -Recurse -Force` on user-provided paths
- Delete files without checking protection lists
- Modify system-critical paths (e.g., `C:\Windows`, `C:\Program Files`)
- Commit code changes unless explicitly requested
- Run destructive operations without `-WhatIf` validation
- Use `$args` as a variable name (reserved in PowerShell)

## ALWAYS Do These

- Use `Remove-SafeItem` helper functions for deletions
- Respect whitelist files (`~\.config\winmole\whitelist`)
- Check protection logic before cleanup operations
- Test with `-WhatIf` mode first
- Validate syntax before suggesting changes
- Use `gh` CLI for GitHub operations when available
- Never commit code unless explicitly requested by user

---

## Quick Reference

### Build Commands

```powershell
# Build Go binaries
.\scripts\build.ps1

# Build specific binary
go build -o bin/analyze.exe ./cmd/analyze
go build -o bin/status.exe ./cmd/status

# Validate PowerShell scripts
.\scripts\build.ps1 validate

# Clean build artifacts
.\scripts\build.ps1 clean
```

### Test Commands

```powershell
# Run full test suite
Import-Module Pester -MinimumVersion 5.0
Invoke-Pester -Path .\tests\ -ExcludeTag Integration

# Run specific test
Invoke-Pester -Path .\tests\WinMole.Tests.ps1

# PowerShell syntax check (all scripts)
Get-ChildItem -Path . -Filter *.ps1 -Recurse | ForEach-Object {
    $null = [System.Management.Automation.Language.Parser]::ParseFile($_.FullName, [ref]$null, [ref]$errors)
    if ($errors) { Write-Host "Errors in $($_.Name): $errors" }
}
```

### Development Commands

```powershell
# Test cleanup in dry-run mode
$env:WINMOLE_DRY_RUN = 1
.\winmole.ps1 clean

# Enable debug logging
$env:WINMOLE_DEBUG = 1
.\winmole.ps1 clean

# Test Go tool directly
go run ./cmd/analyze
go run ./cmd/status
```

---

## Architecture Quick Map

```
winmole/
├── winmole.ps1           # Main CLI entrypoint (menu + routing)
├── install.ps1           # Installer with PATH management
├── bin/                  # Command entry points
│   ├── clean.ps1         # Deep cleanup orchestrator
│   ├── uninstall.ps1     # App removal with leftover detection
│   ├── optimize.ps1      # Service refresh + optimization
│   ├── purge.ps1         # Developer artifact cleanup
│   ├── analyze.ps1       # Disk usage explorer wrapper
│   ├── status.ps1        # System health dashboard wrapper
│   ├── analyze.exe       # Compiled Go TUI
│   └── status.exe        # Compiled Go TUI
├── lib/                  # Reusable PowerShell logic
│   ├── core/             # base.ps1, log.ps1, file_ops.ps1, ui.ps1, common.ps1
│   └── clean/            # Cleanup modules (user.ps1, dev.ps1, system.ps1)
├── cmd/                  # Go applications
│   ├── analyze/          # Disk analysis tool
│   └── status/           # Real-time monitoring
├── scripts/              # Build and test automation
│   └── build.ps1         # Main build script
└── tests/                # Pester tests
    └── WinMole.Tests.ps1
```

**Decision Tree**:

- User cleanup logic -> `lib/clean/<module>.ps1`
- Command entry -> `bin/<command>.ps1`
- Core utils -> `lib/core/<util>.ps1`
- Performance tool -> `cmd/<tool>/*.go`
- Tests -> `tests/*.Tests.ps1`

### Language Stack

- **PowerShell 5.1**: Core cleanup and system operations (`lib/`, `bin/`)
- **Go**: Performance-critical TUI tools (`cmd/analyze/`, `cmd/status/`)
- **Pester**: Unit and integration testing (`tests/`)

---

## Code Style Guidelines

### PowerShell Scripts

- **Indentation**: 4 spaces
- **Variables**: `$PascalCase` for script-level, `$camelCase` for local
- **Functions**: `Verb-Noun` format (e.g., `Remove-SafeItem`, `Get-CacheSize`)
- **Parameters**: Use `[Parameter()]` attributes with proper types
- **Quoting**: Use double quotes for strings with variables, single for literals
- **Comparison**: Use `-eq`, `-ne`, `-gt` etc., not `==`, `!=`

### Go Code

- **Formatting**: Follow standard Go conventions (`gofmt`, `go vet`)
- **Build tags**: Use `//go:build windows` for Windows-specific code
- **Error handling**: Never ignore errors, always handle them explicitly

### Comments

- **Language**: English only
- **Focus**: Explain "why" not "what" (code should be self-documenting)
- **Safety**: Document safety boundaries explicitly

---

## Key Helper Functions

### Safety Helpers (lib/core/file_ops.ps1)

- `Remove-SafeItem -Path <path>`: Safe deletion with validation
- `Remove-OldFiles -Path <path> -Days <n>`: Delete files older than N days
- `Remove-EmptyDirectories -Path <path>`: Clean empty folders
- `Test-ProtectedPath -Path <path>`: Check if path is system-protected


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bhadraagada/winmole](https://github.com/bhadraagada/winmole) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
