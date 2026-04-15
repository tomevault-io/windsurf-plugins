---
trigger: always_on
description: This repository is a collection of reusable PowerShell scripts organized by technology area (Dataverse, EntraID, Files, PowerShell, SharePoint, etc.). Scripts follow a consistent structure and, when they call authenticated APIs, are split into two versions: a **base script** and a **WithAuth wrapper**.
---

# PowerShell Repository — Copilot Instructions

## Overview

This repository is a collection of reusable PowerShell scripts organized by technology area (Dataverse, EntraID, Files, PowerShell, SharePoint, etc.). Scripts follow a consistent structure and, when they call authenticated APIs, are split into two versions: a **base script** and a **WithAuth wrapper**.

---

## Project Structure

```
<TechnologyArea>/
    ScriptName.ps1              # Base script (accepts AccessToken or handles its own auth)
    ScriptNameWithAuth.ps1      # Auth wrapper that acquires a token then calls the base script
EntraID/
    GetAccessTokenDeviceCode.ps1  # Shared device-code-flow auth helper
```

- Group scripts into folders by technology/service (e.g., `Dataverse/`, `SharePoint/`, `Files/`, `PowerShell/`).
- Each folder should contain related scripts that target that technology.

---

## Script Conventions

### Comment-Based Help

Every script **must** start with a `<# ... #>` comment-based help block containing at minimum:

| Section | Required | Notes |
|---|---|---|
| `.SYNOPSIS` | Yes | One-line summary of what the script does. |
| `.DESCRIPTION` | Yes | Detailed explanation of behavior, parameters, and any prerequisites. |
| `.PARAMETER` | Yes | One entry per parameter — describe purpose, valid values, and defaults. |
| `.EXAMPLE` | Yes | At least one realistic usage example with explanation text. |
| `.AUTHOR` | Optional | `Rick Wilson` when included. |
| `.NOTES` | Optional | Prerequisites, module installs, security notes, version info. |

### Parameters

- Define parameters using a `param( )` block immediately after the help comment.
- Use `[Parameter(Mandatory = $true)]` for required parameters.
- Use `[ValidateSet()]` for parameters with a fixed set of values (e.g., `Environment`).
- Provide sensible defaults where appropriate (e.g., `$Environment = "Public"`).
- Use strong typing (`[string]`, `[string[]]`, `[switch]`, `[bool]`).
- For Dataverse/API scripts, the base script **always** takes `[string]$OrganizationUrl` and `[string]$AccessToken` as parameters.

### Code Style

- Use descriptive variable names in PascalCase (e.g., `$AccessToken`, `$OrganizationUrl`).
- Use `Write-Host` with `-ForegroundColor` for user-facing status messages (Cyan for info, Green for success, Yellow for warnings, Red for errors).
- Use `Write-Error` for fatal errors and `Write-Warning` for non-fatal issues.
- Use `Write-Output` or `return` for pipeline-friendly output.
- Extract reusable logic into local functions within the script when needed.
- Use hashtable splatting (`@params`) for calls with many parameters.

---

## Authentication Pattern — Two-Script Approach

When a script calls an authenticated API (Dataverse Web API, Microsoft Graph, etc.), create **two scripts**:

### 1. Base Script (`ScriptName.ps1`)

- Accepts `$AccessToken` as a parameter — it does **not** handle authentication itself.
- Contains all the business logic (API calls, data processing, output).
- Can be called standalone by anyone who already has a token (e.g., from another auth flow, a pipeline, or a service principal).
- Sets up HTTP headers using the provided token:

```powershell
$headers = @{
    "Authorization"    = "Bearer $AccessToken"
    "Content-Type"     = "application/json"
    "OData-MaxVersion" = "4.0"
    "OData-Version"    = "4.0"
}
```

- Uses `Invoke-RestMethod` for API calls.

### 2. Auth Wrapper Script (`ScriptNameWithAuth.ps1`)

- Handles token acquisition, then delegates to the base script.
- Accepts the **same parameters** as the base script **except** `$AccessToken`, and **adds** these auth parameters:

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `$TenantId` | `[string]` | Yes | — | Azure AD tenant ID |
| `$ClientId` | `[string]` | Yes | — | App registration client ID |
| `$Environment` | `[string]` | No | `"Public"` | Azure cloud: `Public`, `GCC`, `GCCH`, `DoD` |

- Acquires a token by calling the shared auth helper:

```powershell
$accessToken = & ..\EntraID\GetAccessTokenDeviceCode.ps1 `
    -TenantId $TenantId `
    -ClientId $ClientId `
    -Scope "$OrganizationUrl/user_impersonation" `
    -Environment $Environment
```

- Then calls the base script, passing the token and all other parameters:

```powershell
$response = & .\ScriptName.ps1 -OrganizationUrl $OrganizationUrl -AccessToken $accessToken -OtherParam $OtherParam
Write-Output $response
```

- For scripts with many parameters, use splatting to forward them cleanly:

```powershell
$scriptParams = @{
    OrganizationUrl = $OrganizationUrl
    AccessToken     = $accessToken
    OutputFormat    = $OutputFormat
}
# conditionally add optional params
if ($Tables) { $scriptParams.Tables = $Tables }

$mainScript = Join-Path $scriptDir "ScriptName.ps1"
$results = & $mainScript @scriptParams
return $results
```

- Validates the token was acquired before proceeding:

```powershell
if (-not $accessToken) {
    Write-Error "Failed to acquire access token."
    exit 1
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rwilson504) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
