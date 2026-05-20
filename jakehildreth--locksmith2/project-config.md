---
trigger: always_on
description: > **Priority order:** Code Style > TDD Workflow > Detection Pattern > Module Structure > Git Workflow
---


# GitHub Copilot Instructions for Locksmith2

> **Priority order:** Code Style > TDD Workflow > Detection Pattern > Module Structure > Git Workflow

## Project Overview

**Locksmith2** is an AD CS security scanner for AD Admins, Defensive Security Professionals, and Security Researchers. It detects and remediates AD CS vulnerabilities (ESC techniques) in Active Directory environments.

Detected techniques (as of 2026.5.14): `ESC1`, `ESC2`, `ESC3c1`, `ESC3c2`, `ESC4a`, `ESC4o`, `ESC5a`, `ESC5o`, `ESC6`, `ESC7a`, `ESC7m`, `ESC8`, `ESC9`, `ESC11`, `ESC13`, `ESC15`, `ESC16`, `Auditing`, `SchemaV1`

**Compatibility:** All code must run in both PS 5.1 and PS 7.x. Never use PS 7-only syntax (`??`, `?:`, named hashtables in class properties, etc.). When in doubt, test in `powershell.exe` (5.1) first.

---

## Code Style

See [instructions/PowersHell.instructions.md](instructions/PowersHell.instructions.md) for detailed PS guidelines. Key rules:

- **Braces:** OTBS — opening brace on same line
- **Indentation:** 4 spaces
- **Casing:** PascalCase for functions/params/public vars; camelCase for private vars
- **No aliases** in scripts (`Get-ChildItem` not `gci`, `Where-Object` not `?`, etc.)
- **Quotes:** single for literals, double for string expansion
- **Continuation:** splatting over backtick
- **CmdletBinding:** `[CmdletBinding()]` on all advanced functions
- **Errors:** `$PSCmdlet.WriteError()` over `Write-Error`; `$PSCmdlet.ThrowTerminatingError()` over `throw`
- **ShouldProcess** for any function that modifies system state
- **Output:** return objects, not text; `PSCustomObject` preferred; `Write-Host` only for UI
- **Pipeline:** `ValueFromPipeline`, Begin/Process/End blocks where appropriate
- **Help:** comment-based help on all public functions — `.SYNOPSIS`, `.DESCRIPTION`, `.PARAMETER`, `.EXAMPLE`, `.OUTPUTS`, `.NOTES`
- **PS 5.1 compatibility:** no null-coalescing (`??`), no ternary (`?:`), no `[nullable]<T>` shorthand

---

## Module Structure

```
Locksmith2/
├── .github/instructions/        # Copilot instructions
├── Build/                       # Build scripts
├── Classes/                     # PS class definitions (loaded via ScriptsToProcess)
│   ├── LS2AdcsObject.ps1        # Main data class for all AD CS objects
│   ├── LS2Issue.ps1             # Issue/finding class
│   └── LS2Principal.ps1        # Principal/identity class
├── Private/
│   ├── Convert/                 # SID/NTAccount/identity conversion
│   ├── Data/                    # ESCDefinitions.ps1, DangerousAces.ps1, etc.
│   ├── Get/                     # LDAP/ADSI query functions
│   ├── Initialize/              # Store initialization (Initialize-LS2Scan, etc.)
│   ├── New/                     # Factory functions
│   ├── Set/                     # Property enrichment functions (Set-CA*, Set-Template*)
│   ├── Test/                    # Boolean test functions
│   ├── UI/                      # Display/formatting
│   └── Utility/                 # General helpers
├── Public/                      # Exported cmdlets
│   ├── Find-LS2VulnerableCA.ps1
│   ├── Find-LS2VulnerableTemplate.ps1
│   ├── Find-LS2VulnerableObject.ps1
│   ├── Find-LS2RiskyPrincipal.ps1
│   ├── Invoke-Locksmith2.ps1
│   ├── New-LS2Dashboard.ps1
│   ├── Get-LS2Stores.ps1
│   └── Set-LS2Forest.ps1
├── Tests/                       # Pester test files (mirror Private/Public structure)
├── Locksmith2.psd1
└── Locksmith2.psm1
```

**One function per file. File name must match function name exactly.**

---

## Data-Driven Detection Pattern

All ESC/technique definitions live in `Private/Data/ESCDefinitions.ps1` using a PowerShell `data {}` constrained language block.

### ESCDefinitions.ps1 structure

```powershell
$script:ESCDefinitions = data {
    @{
        ESC1 = @{
            Technique      = 'ESC1'
            Conditions     = @(
                @{ Property = 'SomeProperty'; Value = $true }
                @{ Property = 'AnotherProperty'; Value = $false }
            )
            EnrolleeProperties = @('DangerousEnrollee')  # optional
            IssueTemplate  = 'Description with $(TemplateName) or $(CAFullName) placeholders'
            FixTemplate    = 'certutil -config $(CAFullName) ...'
            RevertTemplate = 'certutil -config $(CAFullName) ...'
        }
    }
}
```

**Variable expansion in templates:** `$(PropertyName)` — expanded at issue-generation time using the object's property values.

### Adding a new technique — required changes checklist

When adding any new technique, update ALL of these:

1. **`Private/Data/ESCDefinitions.ps1`** — add the technique entry
2. **`Public/Find-LS2VulnerableTemplate.ps1`** or **`Find-LS2VulnerableCA.ps1`** — add to `[ValidateSet(...)]` and to the `$templateTechniques`/`$caTechniques` array; add a new `elseif` branch only if it needs non-standard logic (most techniques use the generic condition-based path)
3. **`Private/Initialize/Initialize-LS2Scan.ps1`** — add to `$templateTechniques` or `$caTechniques` array
4. **`Public/Invoke-Locksmith2.ps1`** — add to `$techniques` array
5. **`Tests/Private/Data/ESCDefinitions.Tests.ps1`** — add to `$RequiredTechniques`, `$TechniquesWithConditions` (if applicable), and `$AllTechniques`

### Non-standard branches


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jakehildreth/Locksmith2](https://github.com/jakehildreth/Locksmith2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
