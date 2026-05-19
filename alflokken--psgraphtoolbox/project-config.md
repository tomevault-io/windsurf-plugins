---
trigger: always_on
description: PowerShell toolkit for Microsoft Entra ID operations via Microsoft Graph API. Uses `Microsoft.Graph` SDK with custom wrappers for pagination, batching, and throttle handling.
---

# PSGraphToolbox Graph Toolkit - AI Coding Instructions

## Project Overview
PowerShell toolkit for Microsoft Entra ID operations via Microsoft Graph API. Uses `Microsoft.Graph` SDK with custom wrappers for pagination, batching, and throttle handling.

**Core Philosophy**: Minimal dependencies, CLM-compatible, PowerShell 5.1+, pipeline-friendly.

---

## ⚠️ CRITICAL: PowerShell 5.1 Constrained Language Mode

**ALL code MUST work in PowerShell 5.1 Constrained Language Mode (CLM).**

### ❌ Prohibited
- Static .NET methods: `[System.IO.File]::ReadAllText()`, `[Guid]::Parse()`
- No [PSCustomObject]@{}: Type accelerator blocked, use New-Object PSObject + Add-Member
- `Add-Type`: Cannot load custom C#
- `[scriptblock]::Create()`: Dynamic script blocks blocked
- COM objects: `New-Object -ComObject`
- Most `New-Object` calls for non-approved types

### ✅ Safe Alternatives
```powershell
# String operations - use operators
$text -replace "a", "b"          # Not: $text.Replace("a", "b")
$text -split ","                 # Not: $text.Split(",")

# GUID operations
[System.Guid]$guid = $string     # Not: [Guid]::Parse($string)

# Date operations  
Get-Date $string                 # Not: [DateTime]::Parse($string)
(Get-Date).ToUniversalTime()     # Not: [DateTime]::UtcNow

# File operations
Get-Content -Path $path -Raw     # Not: [System.IO.File]::ReadAllText()
Set-Content -Path $path -Value $content
Test-Path -Path $path

# HTML encoding
$text -replace "&","&amp;" -replace "<","&lt;" -replace ">","&gt;"
```

---

## ⚠️ AI-Generated Code Marking

**Mark ALL AI-generated code clearly.**

```powershell
# Entire function
<#
.DESCRIPTION
[AI-GENERATED] This function was generated using Claude Sonnet 4.5.
Requires scopes: User.Read.All
#>

# Code block
# [AI-GENERATED START] - Batch processing logic
$chunks = Split-ArrayIntoChunks -Enumerable $items -ChunkSize 20
# [AI-GENERATED END]

# Inline
$sorted = $data | Sort-Object Date  # [AI-GENERATED] sorting logic
```

---

## Architecture Quick Reference

### Core Layer (`core/`)
- **`Invoke-GtGraphRequest`** - Graph API wrapper with auto-pagination, OData support
- **`Invoke-GtGraphBatchRequest`** - Batch processor (20/batch), auto-retry on throttle
- **`Sync-GtGraphResourceDelta`** - Delta sync with state persistence
- **`Get-GtGraphDirectoryObjectsByIds`** - Bulk ID resolver (chunks to 1000)

### Helpers (`helpers/`)
Internal utilities (not exported):
- **`Get-IdFromInputObject`** - Normalizes UPN/GUID/object to ID
- **`isValidGuid`** / **`isValidUserPrincipalName`** - Validation filters
- **`New-GtRequestUri`** - OData URI builder

### Utilities (`utilities/`)
- **`Split-ArrayIntoChunks`** - Array chunking (alias: `chunk`)
- **`Export-GtHtmlReport`** - Bootstrap HTML report generator

---

## Naming Conventions

- **Prefix**: `Gt` for all public functions (e.g., `Get-GtUser`, `Find-GtGroup`)
- **Verbs**: `Get-` (retrieval), `Find-` (search), `Add-`/`Remove-` (mutations), `Invoke-` (actions)
- **Parameters**: PascalCase, full names (no abbreviations)

---

## Key Patterns

### 1. Input Flexibility
Accept UPN, GUID, or objects with `id`/`userPrincipalName`:

```powershell
function Get-GtUser {
    param(
        [Parameter(Mandatory, ValueFromPipeline, Position = 0)]
        [object]$InputObject
    )
    process {
        $userId = Get-IdFromInputObject -InputObject $InputObject
        Invoke-GtGraphRequest -resourcePath "users/$userId"
    }
}

# All valid:
Get-GtUser "user@contoso.com"
Get-GtUser "guid-here"
$user | Get-GtUser
```

### 2. Always Use Invoke-GtGraphRequest
```powershell
# ✅ Correct - auto-pagination, OData params
$users = Invoke-GtGraphRequest -resourcePath "users" `
    -select "id,displayName" `
    -filter "accountEnabled eq true"

# ❌ Wrong - manual pagination required
$users = Invoke-MgGraphRequest -Uri "..."
```

### 3. OData Parameters with Splatting
```powershell
$params = @{
    resourcePath = "users"
    apiVersion   = "beta"  # or "v1.0" (default)
    select       = "id,displayName,signInActivity"
    filter       = "userType eq 'Member'"
    expand       = "manager(`$select=displayName)"
}
$users = Invoke-GtGraphRequest @params
```

### 4. Batch Processing
```powershell
$chunks = Split-ArrayIntoChunks -Enumerable $userIds -ChunkSize 20
foreach ($chunk in $chunks) {
    $requests = $chunk | ForEach-Object {
        @{ id = $_; method = "GET"; url = "users/$_" }
    }
    $results = Invoke-GtGraphBatchRequest -Requests $requests
}
```

---

## Comment-Based Help Guidelines

**Scale help to function complexity.** Simple functions need minimal help; complex functions need detailed examples.

### Critical Formatting Rule
**Comment-based help keywords (e.g., .EXAMPLE) and their content must be at the SAME indentation level - no tabs/spaces before the description text.**

```powershell
<#
.SYNOPSIS
Retrieves a user from Microsoft Entra ID

.DESCRIPTION
Accepts flexible input (UPN, GUID, or object) and returns user details.
Requires scopes: User.Read.All

.PARAMETER InputObject
User identifier (UPN, GUID, or object with id/userPrincipalName)

.EXAMPLE
Get-GtUser "user@contoso.com"
Retrieves user by UPN

.EXAMPLE

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alflokken/PSGraphToolbox](https://github.com/alflokken/PSGraphToolbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
