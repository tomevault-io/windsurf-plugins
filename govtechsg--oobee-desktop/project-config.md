---
trigger: always_on
description: This document defines the expected UI behavior for scan type handling in:
---

# Home Page Scan-Type UI Logic

This document defines the expected UI behavior for scan type handling in:

- `src/MainWindow/HomePage/InitScanForm.jsx`
- `src/MainWindow/HomePage/AdvancedScanOptions.jsx`
- `src/MainWindow/HomePage/index.jsx` (scan submission + validation behavior)

## Scan Types

From `src/common/constants.js`:

1. `Intelligent crawler`
2. `Sitemap crawler`
3. `Website crawler`
4. `Custom flow`
5. `Local file`

---

## Core Rules

- Avoid index-based assumptions when UI options are filtered.
- Prefer explicit scan type labels for conditional rendering in advanced options.
- URL/File mode switching must preserve a valid non-file scan type.
- `Local file` must only be used as file-mode scan type, not cached as URL-mode fallback.

---

## URL vs File Toggle Behavior

### URL mode
- Scan Type options include all except `Local file`.
- URL/File toggle is:
  - shown for all URL-mode scan types except `Custom flow`
  - hidden for `Custom flow`

### File mode
- Scan Type options must be exactly:
  1. `Local file`
  2. `Sitemap crawler`
  3. `Custom flow`
- URL/File toggle must remain visible so user can switch back to URL.
- Default selection on URL → File toggle is `Local file`.

### Restoring File → URL mode
- Restore the last cached non-file scan type.
- If cached non-file scan type is `Local file`, fallback to `Intelligent crawler`.
- Do not cache `Local file` as `cachedNonFileScanType`.

---

## Control Visibility Matrix

| Control | Intelligent | Sitemap | Website | Custom flow | Local file |
|---|---|---|---|---|---|
| URL/File toggle (URL mode) | Show | Show | Show | Hide | N/A |
| URL/File toggle (File mode) | N/A | N/A | N/A | N/A | Show |
| Capped at N pages | Show | Show | Show | Hide | Hide |
| File Type dropdown | Show | Show | Show | Hide | Show |
| Allow subdomains for scans | Show | Hide | Show | Hide | Hide |
| Slow Scan Mode | Show | Show | Show | Hide | Hide |
| Adhere to robots.txt | Show | Hide | Show | Hide | Hide |

---

## File-Type Handling in File Mode

- `Local file`: allow webpage/pdf extensions (`.html`, `.htm`, `.shtml`, `.xhtml`, `.pdf`)
- `Sitemap crawler`: allow sitemap extensions (`.xml`, `.txt`)
- `Custom flow`: allow webpage/pdf extensions (`.html`, `.htm`, `.shtml`, `.xhtml`, `.pdf`) (same as website crawl)

---

## Submission & Validation Rules

- In file mode, **do not force** `scanType` to `Local file` during submit.
- Submit the currently selected file-mode scan type (`Local file` / `Sitemap crawler` / `Custom flow`).
- Persist the actual selected `scanType`; use `isFileOptionChecked` to determine URL vs FILE mode.
- In `HomePage/index.jsx` validation:
  - `Sitemap crawler`: allow `http(s)://` and `file://`
  - `Local file`: allow `file://` only
  - `Custom flow`: allow `http(s)://` and `file://`

---

## Guardrails for Future Changes

- If scan type list order changes, behavior must not break.
- Keep conditional logic label-based where options can be filtered.
- When changing file-mode options, retest:
  - URL → File toggle
  - File → URL toggle
  - dropdown persistence
  - visibility matrix above
  - file-mode submit payload uses selected scan type (not forced `Local file`)
  - file-mode `Custom flow` with valid `file://` path does not show `Invalid URL.`

---

# Auto-Update System

This document defines the update mechanism behavior in:

- `public/electron/updateManager.js`
- `public/electron/main.js`

## Platform-Specific Update Flows

### Windows Update Flow

**Uses Installer-Based Updates:**

1. **PowerShell Check**: Verifies PowerShell is available (line 321)
2. **Download**: Uses PowerShell `System.Net.WebClient` to download installer package
3. **Extract**: Uses `Expand-Archive` to unzip to `%APPDATA%\Oobee\oobee-desktop-windows\`
4. **Launch Installer**: Spawns `Oobee-setup.exe` as detached process
5. **Exit**: App exits, installer handles replacement

**Key Points:**
- Backend is packaged in the installer (no separate backend update)
- Installer handles privilege elevation automatically
- Uses `installerLaunched` event to trigger app exit
- No custom relaunch logic needed (installer handles it)

**PowerShell Script:**
```powershell
$webClient = New-Object System.Net.WebClient
$webClient.DownloadFile("${downloadUrl}", "${resultsPath}\\oobee-desktop-windows.zip")
Expand-Archive -Path "${resultsPath}\\oobee-desktop-windows.zip" -DestinationPath "${resultsPath}\\oobee-desktop-windows" -Force
```

### macOS Update Flow

**Uses In-Place Binary Replacement:**

1. **Version Check**: Compares current version with latest release
2. **Download**: Downloads update package to temp directory
3. **Privilege Check**: Tests if app location is writable
4. **Installation**: Replaces app bundle (with elevation if needed)
5. **Relaunch**: Uses macOS `open -n` command to launch updated app

---

## macOS Implementation Details

### Privilege Elevation (`updateManager.js`)

**Automatic Detection:**
- Checks if app location is writable using `fs.accessSync()` with `W_OK` flag
- Tests both parent directory and app bundle for write permissions
- Only elevates if **any** path is not writable

**Admin By Request Support:**
- Uses native macOS `osascript` with "administrator privileges"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GovTechSG/oobee-desktop](https://github.com/GovTechSG/oobee-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
