---
trigger: always_on
description: This repository contains custom [Maester](https://www.maester.dev) tests for Microsoft 365 tenants. Tests are PowerShell/Pester-based and validate user attributes, formatting rules, and organizational standards against Microsoft Graph API data.
---

# Custom Maester Tests - AI Agent Instructions

## Repository Purpose

This repository contains custom [Maester](https://www.maester.dev) tests for Microsoft 365 tenants. Tests are PowerShell/Pester-based and validate user attributes, formatting rules, and organizational standards against Microsoft Graph API data.

## Core Concepts

### Test Structure (3-File Pattern)

**Every test requires exactly 3 files with matching names:**

1. **`Test-ContosoXXXX.ps1`** - PowerShell function implementing test logic
2. **`Test-ContosoXXXX.tests.ps1`** - Pester test file that sources and runs the function
3. **`Test-ContosoXXXX.md`** - Documentation with test description, company policy, remediation steps

### Test Categories

**Entra ID Tests** - Organized in `tests/Entra/`:

- **Attributes/** - Tests for missing required user attributes (business phone, manager, department, etc.)
- **Formatting/** - Tests validating attribute format compliance (phone numbers, postal codes, email formats)
- **Validating/** - Tests checking attributes against allowed values defined in `validation.json`

**Teams Tests** - Organized in `tests/Teams/`:

- **Configuration/** - Tests for Teams client configuration and security settings (third-party storage, external access, etc.)

## File Conventions

### Test Function (.ps1)

```powershell
function Test-ContosoUsers{Purpose} {
    param(
        [string] $ValidatingPath = ("$PSScriptRoot/validation.json")  # Only for Validating tests
    )
    $result = $true
    try {
        # Get users from Graph API
        $users = Invoke-MtGraphRequest -RelativeUri "users" -Filter "userType eq 'Member'" -Select "displayName,{attributes}"
        $invalidUsers = @()

        foreach ($user in $users) {
            # ALWAYS skip sync account
            if ($user.displayName -eq "On-Premises Directory Synchronization Service Account") {
                continue
            }
            
            # Test logic here
            if (# validation fails) {
                $result = $false
                $invalidUsers += $user
            }
        }

        if ($result) {
            $TestResults = "Well done! {Success message}"
        } else {
            $TestResults = "{Failure message}:`n%TestResult%"
        }
        
        Add-MtTestResultDetail -Result $TestResults -GraphObjects $invalidUsers -GraphObjectType Users
    } catch {
        $result = $false
        Write-Error $_.Exception.Message
    }
    return $result
}
```

### Pester Test (.tests.ps1)

```powershell
BeforeAll {
    . $PSScriptRoot/Test-ContosoUsers{Purpose}.ps1
}

Describe "Contoso" -Tag "{TEST_ID}", "Entra", "CustomTests", "{Category}", "Users" {
    It "{TEST_ID}: {Description} - {Expectation}" {
        $result = Test-ContosoUsers{Purpose}
        $result | Should -Be $true -Because "{Reason}"
    }
}
```

### Test IDs

**Entra ID Test IDs** - Format: `ENTRA.{Category}.{Number}[.{Variant}]`

- **UA** - User Attributes (e.g., `ENTRA.UA.1001`)
- **UF** - User Formatting (e.g., `ENTRA.UF.1002.T01.USA`)
- **UV** - User Validation (e.g., `ENTRA.UV.1003`)

**Teams Test IDs** - Format: `TEAMS.{Category}.{Number}`

- **TC** - Teams Configuration (e.g., `TEAMS.TC.5002`)

### Markdown Documentation (.md)

```markdown
This test checks {what the test validates}.

Contoso's company policy requires {business requirement}. This is important for {reason}.

**To remediate this issue:**

- Identify the users {with the issue}.
- Raise a ticket in Service Now using [Form: {Title}](https://contoso.service-now.com/{path}) to request {action}.
  - 🔺 If this is not actioned in three days, escalate to {role}.

**Learn more:**

- [{Link Title}](https://contoso.service-now.com/{path})

<!--- Results --->

%TestResult%
```

## Validation Configuration

`tests/Entra/Validating/validation.json` contains allowed values for validation tests:

```json
{
  "validTitles": [{"title": "CEO"}, ...],
  "validDepartments": [{"department": "IT"}, ...],
  "validLocations": [{
    "streetAddress": "...",
    "city": "...",
    "state": "...",
    "zipCode": "...",
    "country": "...",
    "businessPhones": "...",
    "companyName": "..."
  }, ...]
}
```

Validating tests load this file to check user attributes against allowed values.

## Creating New Tests

### Step 1: Determine Test ID and Category

- Attributes test: Check for missing required fields
- Formatting test: Validate field formats with regex patterns
- Validating test: Check against allowed values from validation.json

Assign next available test ID in the category (check existing tests).

### Step 2: Create All 3 Files

Use the templates above. Ensure:
- Function name matches file names
- Test ID is consistent across all files
- Appropriate tags are included in Pester test
- Markdown includes company policy and remediation steps

### Step 3: Common Patterns

**Always skip sync account:**
```powershell
if ($user.displayName -eq "On-Premises Directory Synchronization Service Account") {
    continue
}
```

**Common exclusions:**
```powershell
if ($user.jobTitle -eq "CEO") {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DevClate/Custom-Maester-Tests](https://github.com/DevClate/Custom-Maester-Tests) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
