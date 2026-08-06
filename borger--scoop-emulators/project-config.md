---
trigger: always_on
description: - **Testing Manifests**: When testing, verifying, or validating Scoop manifests, always use the helper scripts located in the `bin` directory of the workspace, such as:
---

# Scoop Emulators Bucket - AI Instructions

## Agent Rules & Guidelines

- **Testing Manifests**: When testing, verifying, or validating Scoop manifests, always use the helper scripts located in the `bin` directory of the workspace, such as:
  - `bin/check-manifest-install.ps1` to test installation and verification of a manifest.
  - `bin/checkver.ps1` to test checkver update patterns.
  - `bin/test.ps1` to run general Pester tests for the bucket.

- **Checkver & Autoupdate Guidelines**:
  - `autoupdate` URLs can safely use regex capture groups like `$matchBuild` and `$matchCommit`. The local `check-manifest-install.ps1` script natively handles these by falling back to testing the hardcoded architecture URLs, preventing false-positive 404 test failures.
  - When scraping sites for `checkver`, always prefer official JSON APIs over HTML parsing (e.g., use `api.github.com` or official updater APIs). Sites like Dolphin Emulator use Cloudflare, which blocks `Invoke-WebRequest` HTML parsing, resulting in 403 Forbidden errors.
  - Prefer using Scoop's native GitHub version detection (`"github": "https://github.com/owner/repo"`) instead of manually scraping GitHub release pages with regex. It is more robust against release structure changes.
  - For projects that are ports or recompilations (e.g. recomp projects), archive structures (like folder naming inside `.zip` or `.rar`) and binary executable names might change between releases. Always extract and verify the actual filenames/directory tree of the latest asset to update `extract_dir` and the `bin` entry accordingly.
## Core Requirements

This bucket contains emulator manifests validated by 3 tests:
1. **checkver** - Detects latest version
2. **check-autoupdate** - Validates autoupdate config and URLs
3. **check-manifest-install** - Tests manifest installation

All three must pass. PRs auto-merge on pass, escalate to @beyondmeat on failure.

---

## PowerShell Scripts

**Location:** `bin/` directory

**Key scripts:**
- `checkver.ps1` - Detect latest version
- `check-autoupdate.ps1` - Validate autoupdate config
- `check-manifest-install.ps1` - Test manifest installation
- `update-manifest.ps1` - Auto-update version and hashes
- `autofix-manifest.ps1` - **Intelligent manifest repair** with version scheme detection and recovery
- `validate-and-merge.ps1` - Full PR validation pipeline

**Usage:**
```powershell
# Single manifest validation
.\bin\checkver.ps1 -Dir bucket -App appname
.\bin\check-autoupdate.ps1 -ManifestPath bucket/app.json
.\bin\check-manifest-install.ps1 -ManifestPath bucket/app.json

# Update and auto-fix
.\bin\update-manifest.ps1 -ManifestPath bucket/app.json -Update
.\bin\autofix-manifest.ps1 -ManifestPath bucket/app.json
```

---

## PowerShell Compatibility (5.1 & 7.x)

**Critical Rules:**

1. **No Special Characters** - Use ASCII only
   - USE: `[OK]`, `[FAIL]`, `[WARN]`, `[SKIP]`, `[INFO]`
   - AVOID: emoji and Unicode checkmarks

2. **File Encoding** - UTF-8 WITHOUT BOM
    ```powershell
    $utf8NoBom = New-Object System.Text.UTF8Encoding $false
    [System.IO.File]::WriteAllText($path, $content, $utf8NoBom)
    ```
   - DO NOT use `Set-Content` or `Out-File` (they add BOM)
   - DO NOT use `UTF8Encoding($true)`

3. **Progress Preference** - Use variable, not parameter
   ```powershell
   $ProgressPreference = 'SilentlyContinue'
   Invoke-WebRequest -Uri $url -OutFile $file
   ```

4. **Avoid Version-Specific Features**
   - SKIP: `??` null-coalescing operator (PS 7.x only)
   - SKIP: `?.` null-conditional (PS 7.x only)
   - USE: `if` statements instead

5. **JSON Trailing Newline** - Always add `+ "`n"`
    ```powershell
    $utf8NoBom = New-Object System.Text.UTF8Encoding $false
    [System.IO.File]::WriteAllText($path, $json + "`n", $utf8NoBom)
    ```

6. **Windows 11 Environment** - Commands should be windows compatible
   - AVOID: Unix utilities like `head`, `tail`, `grep`, `sed`, `awk`
   - USE: PowerShell cmdlets instead:
     - First 5 items: `Select-Object -First 5`
     - Last 5 items: `Select-Object -Last 5`
     - Pattern matching: `Select-String -Pattern pattern`
     - Sorting: `Sort-Object`

---

## File Standards

**All files: UTF-8 WITHOUT BOM, CRLF line endings, trailing newline, NO TRAILING WHITESPACE**

### Critical: UTF-8 WITHOUT BOM
**BOM (Byte Order Mark) breaks GitHub Actions workflows!**
- UTF-8 BOM = `EF BB BF` bytes at start of file
- PowerShell's default `[System.Text.Encoding]::UTF8` **ADDS BOM** (wrong!)
- Use `New-Object System.Text.UTF8Encoding $false` instead (correct!)

**NEVER use:**
```powershell
[System.IO.File]::WriteAllText($path, $content, [System.Text.Encoding]::UTF8)  # ADDS BOM!
```

**ALWAYS use:**
```powershell
$utf8NoBom = New-Object System.Text.UTF8Encoding $false
[System.IO.File]::WriteAllText($path, $content, $utf8NoBom)  # NO BOM!
```

### Additional PS5.1 rules learned
- Always test PowerShell scripts using Windows PowerShell 5.1 (powershell.exe -NoProfile). Do not rely only on PowerShell 7 (pwsh) during development or CI for scripts that must run on Windows host environments.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [borger/scoop-emulators](https://github.com/borger/scoop-emulators) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
