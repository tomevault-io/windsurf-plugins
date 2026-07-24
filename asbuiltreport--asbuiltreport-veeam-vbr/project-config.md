---
trigger: always_on
description: This module generates as-built documentation reports (HTML, Word, Text) for Veeam Backup & Replication (VBR) infrastructure. It plugs into the [AsBuiltReport](https://github.com/AsBuiltReport/AsBuiltReport.Core) framework via `Invoke-AsBuiltReport.Veeam.VBR`, which is the sole public entry point.
---

# Copilot Instructions for AsBuiltReport.Veeam.VBR

## Project Overview

This module generates as-built documentation reports (HTML, Word, Text) for Veeam Backup & Replication (VBR) infrastructure. It plugs into the [AsBuiltReport](https://github.com/AsBuiltReport/AsBuiltReport.Core) framework via `Invoke-AsBuiltReport.Veeam.VBR`, which is the sole public entry point.

**Runtime constraint:** Windows PowerShell 5.1 **only** — PS7/PowerShell Core is not supported. VBR v12+ (Windows/Appliance installs). VBR v13 is explicitly unsupported due to .NET Core changes.

## Lint Command

PSScriptAnalyzer runs automatically on push/PR via `.github/workflows/PSScriptAnalyzer.yml`. To run it locally:

```powershell
Invoke-ScriptAnalyzer -Path .\Src -Recurse -Settings .\.github\workflows\PSScriptAnalyzerSettings.psd1
```

There are no Pester tests and no build script.

## Architecture

```
Invoke-AsBuiltReport.Veeam.VBR.ps1   ← single public function; orchestrates entire report
Src/Private/Report/Get-AbrVbr*.ps1   ← 91 report section functions (one per VBR feature area)
Src/Private/Diagram/Get-Abr*.ps1     ← 56 diagram helper functions
AsBuiltReport.Veeam.VBR.psm1         ← dot-sources all Src/**/*.ps1; exports Public + Diagram + Report
AsBuiltReport.Veeam.VBR.json         ← user-facing config: InfoLevel, HealthCheck, Options schemas
```

The framework calls `Invoke-AsBuiltReport.Veeam.VBR` with `$Target`, `$Credential`, and a parsed `$ReportConfig`. The function imports config into three `$script:` variables (`$script:InfoLevel`, `$script:HealthCheck`, `$script:Options`) and then calls each `Get-AbrVbr*` function in sequence, wrapped in `Section` blocks from PScribo.

## Key Conventions

### Function & Variable Naming

- Report section functions: `Get-AbrVbr<FeatureName>` (e.g., `Get-AbrVbrBackupRepository`)
- Diagram functions: `Get-Abr<DiagramType>` (e.g., `Get-AbrVbrDiagramBackupProxy`)
- Output collection: `$OutObj` (array), individual item: `$inObj` (ordered hashtable)
- Script-scope config: `$script:InfoLevel`, `$script:HealthCheck`, `$script:Options`, `$script:TextInfo`

### Standard Report Section Structure

Every `Get-AbrVbr*` function follows this skeleton:

```powershell
function Get-AbrVbr<Name> {
    [CmdletBinding()]
    param()

    begin {
        Write-PScriboMessage "Discovering Veeam VBR <Name>."
        Show-AbrDebugExecutionTime -Start -TitleMessage '<Name>'
    }

    process {
        try {
            [Array]$Data = Get-VBR<ObjectType> | Sort-Object -Property Name
            if ($Data) {
                $OutObj = @()
                foreach ($Item in $Data) {
                    try {
                        $inObj = [ordered] @{
                            'Name'   = $Item.Name
                            'Status' = $Item.Status
                            # ...
                        }
                        $OutObj += [pscustomobject](ConvertTo-HashToYN $inObj)
                    } catch {
                        Write-PScriboMessage -IsWarning "<Name>: $($_.Exception.Message)"
                    }
                }

                if ($HealthCheck.<Category>.<Section>) {
                    # Apply Set-Style -Style Warning/Critical/OK to $OutObj rows
                }

                Section -Style Heading3 '<Title>' {
                    $OutObj | Table -Name '<Title>'
                }

                if ($InfoLevel.<Category>.<Section> -ge 2) {
                    # Detailed per-item sub-sections
                }
            }
        } catch {
            Write-PScriboMessage -IsWarning "<Name> Section: $($_.Exception.Message)"
        }
    }
    end {
        Show-AbrDebugExecutionTime -End -TitleMessage '<Name>'
    }
}
```

### InfoLevel Gates

InfoLevel values (0–3) come from `$script:InfoLevel.<Category>.<Section>`:

- `0` — section disabled entirely
- `1` — summary table
- `2` — advanced summary (additional columns or sub-tables)
- `3` — full detail (per-item sections, hardware inventory, etc.)

Gate detailed content with:
```powershell
if ($InfoLevel.Infrastructure.BR -ge 2) { ... }
```

### Health Check Highlighting

Health checks are boolean flags from `$script:HealthCheck.<Category>.<Section>`. When enabled, apply PScribo styles to `$OutObj` rows before passing to `Table`:

```powershell
if ($HealthCheck.Infrastructure.BR) {
    $OutObj | Where-Object { $_.'Status' -ne 'OK' } | Set-Style -Style Warning -Property 'Status'
}
```

Available styles: `OK`, `Warning`, `Critical`, `Info`.

### Boolean Display

Always wrap the final `$inObj` hashtable with `ConvertTo-HashToYN` to convert `$true`/`$false` to `"Yes"`/`"No"`:

```powershell
$OutObj += [pscustomobject](ConvertTo-HashToYN $inObj)
```

### Veeam API Usage

- Primary: `Veeam.Backup.PowerShell` cmdlets (`Get-VBRJob`, `Get-VBRBackupRepository`, etc.) — must be loaded on the VBR server
- Low-level: Direct assembly calls for objects not exposed via cmdlets, e.g. `[Veeam.Backup.Core.CBackupJob]::GetAll()` for Nutanix jobs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AsBuiltReport/AsBuiltReport.Veeam.VBR](https://github.com/AsBuiltReport/AsBuiltReport.Veeam.VBR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
