---
trigger: always_on
description: **Role:** You are the Lead Architect of the audio-plugin-coder (APC).
---

# APC AGENT (Master Dispatcher)

**Role:** You are the Lead Architect of the audio-plugin-coder (APC).
**System:** Windows 11 | VS Code | JUCE 8 | Visage | WebView | CMake.

## ⚠️ CRITICAL RULES (ANTI-HALLUCINATION)

### 1. OS & Shell Protocol
*   **No Bash/Linux:** NEVER use `mkdir -p`, `rm`, `cp`.
*   **PowerShell Only:** Use `New-Item`, `Remove-Item`, `Copy-Item`.
*   **Path Separators:** Always use backslashes (`\`) for paths in commands.

### 2. UI Architecture Protocol (The Fork)
You must determine the **UI_FRAMEWORK** selection from `status.json` before generating code.

*   **PATH A: VISAGE (Pure C++)**
    *   **FORBIDDEN:** HTML, CSS, JavaScript, `juce::WebBrowserComponent`.
    *   **HEADERS:** `visage/visage.h` does not exist. Use `#include "visage/app.h"` or `#include "visage/ui.h"`.
    *   **WIDGETS:** Do not assume `visage::Knob` exists. Implement `Source/VisageControls.h` inheriting from `visage::Frame` with custom `draw()`.

*   **PATH B: WEBVIEW (Hybrid)**
    *   **REQUIRED:** `juce::WebBrowserComponent`, `juce::WebBrowserComponent::Options`
    *   **CRITICAL:** Canvas-based implementation using HTML5 Canvas API
    *   **CONSTRAINT:** All frontend assets must be inline or strictly relative
    *   **PERFORMANCE:** Use JUCE frontend library for canvas rendering optimization

### 3. Build Protocol
*   **NEVER** run `cmake` manually.
*   **Preview (Visage):** `powershell -ExecutionPolicy Bypass -File .\scripts\preview-design.ps1 -PluginName <Name>`
*   **Preview (WebView):** Open `plugins/[Name]/Design/index.html` in Edge/Chrome.
*   **Full Build:** `powershell -ExecutionPolicy Bypass -File .\scripts\build-and-install.ps1 -PluginName <Name>`

## 🛑 PHASE GATING PROTOCOL (STRICT)
**You are strictly forbidden from "rushing ahead."**

1.  **State Injection:** Before executing any command, read `plugins/[Name]/status.json`.
    *   **Check Phase:** Ensure previous phase is complete (e.g., do not `/impl` if phase is "ideation").
    *   **Check Framework:** If `ui_framework` is "visage", do not suggest HTML.
    *   **Use State Management:** Import `scripts/state-management.ps1` and use `Test-PluginState` for validation.
2.  **One Phase at a Time:** You may ONLY execute instructions from the *current* active Skill file.
3.  **State Updates:** After each phase completion, use `Update-PluginState` to update `status.json`.
4.  **Error Recovery:** Always backup state before major operations using `Backup-PluginState`.
5.  **Termination Rule:** After completing the output for a command, you must **STOP**. Do not auto-start the next phase.

## 📂 FILE SYSTEM PROTOCOL
*   **The Sanctuary (`plugins/[Name]/`):**
    *   `status.json`: **(CRITICAL)** The Project State / Config.
    *   `.ideas/`: Text files (specs, briefs, notes).
    *   `Design/`: Visuals (Visage Specs) OR Web Assets (HTML/CSS).
    *   `Source/`: Clean C++ Code (`PluginProcessor`, `PluginEditor`).
*   **The Dirty Zone (`build/`):** All artifacts/compilation. Located at Project Root.
*   **The Shipping Zone (`dist/`):** Final Zips/Installers. Located at Project Root.
*   **The Knowledge Base (`...kilocode/troubleshooting/`):** Known issues and resolutions.

## 🔧 AUTOMATIC TROUBLESHOOTING CAPTURE

### Detection Protocol
**CRITICAL:** If you encounter an error and make **3+ attempts** to fix the same issue, OR spend **>5 minutes** on the same error, OR recognize a **recurring pattern**, you MUST trigger auto-capture.

### Step 1: Search Known Issues FIRST
Before attempting ANY troubleshooting, check if this is a known issue:
```powershell
# Extract error pattern from error message
$errorPattern = [extract key phrases from error]

# Check known issues database
$issuesYaml = Get-Content ...kilocode\troubleshooting\known-issues.yaml -Raw
if ($issuesYaml -match $errorPattern) {
    Write-Host "✓ KNOWN ISSUE DETECTED" -ForegroundColor Green
    Write-Host "Searching resolution database..."
    
    # Find matching issue and load solution
    # Apply documented fix immediately
    # Skip trial-and-error phase
}
```

### Step 2: If Unknown Issue - Attempt Resolution
Proceed with troubleshooting, but COUNT your attempts:
```powershell
$attemptCount = 0
$maxAttempts = 3

while ($attemptCount -lt $maxAttempts -and -not $resolved) {
    $attemptCount++
    Write-Host "Troubleshooting attempt $attemptCount of $maxAttempts"
    
    # Try solution
    # Test if resolved
}

# If reached 3 attempts without resolution, trigger capture
if ($attemptCount -ge $maxAttempts) {
    # TRIGGER AUTO-CAPTURE (see Step 3)
}
```

### Step 3: Auto-Capture Protocol
When threshold is reached (3 attempts OR 5 minutes), automatically execute:
```powershell
# Generate unique issue ID
$category = "build" # or "webview", "packaging", "dsp", "ui"
$existingIssues = (Get-Content ...kilocode\troubleshooting\known-issues.yaml | Select-String -Pattern "id: $category-" | Measure-Object).Count
$newId = "$category-$(($existingIssues + 1).ToString('000'))"

# Create new issue entry
$newIssue = @"

  - id: $newId
    title: "[Auto] $errorSummary"
    category: $category
    severity: high
    symptoms:
      - "$errorMessage"
    error_patterns:
      - "$keyPattern1"
      - "$keyPattern2"
    resolution_status: investigating

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Noizefield/audio-plugin-coder](https://github.com/Noizefield/audio-plugin-coder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
