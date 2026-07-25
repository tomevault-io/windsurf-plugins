---
trigger: always_on
description: Text Grab is a Windows-specific .NET 10.0 WPF OCR (Optical Character Recognition) application that extracts text from images using Windows APIs. It provides multiple modes for text capture including full-screen grab, grab frame, edit text window, and quick lookup.
---

# Text Grab - GitHub Copilot Instructions

Text Grab is a Windows-specific .NET 10.0 WPF OCR (Optical Character Recognition) application that extracts text from images using Windows APIs. It provides multiple modes for text capture including full-screen grab, grab frame, edit text window, and quick lookup.

**ALWAYS reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

## CRITICAL: Windows-Only Application

**DO NOT attempt to fully build or run this application on non-Windows platforms.** The application:
- Requires Windows-specific APIs (Windows.Media.Ocr, WPF, WinForms)
- Uses Windows 10 SDK 22621.0 and WindowsAppSDK
- Build packaging fails on Linux/macOS due to Windows-specific MSBuild tasks
- Only dependency restoration and partial compilation validation possible on non-Windows

## Working Effectively

### Prerequisites (Windows Only)
For full development on Windows:
- Windows 10/11 with Windows 10 SDK 22621.0
- Visual Studio 2022 with workloads:
  - "Universal Windows Platform Development" 
  - ".NET desktop development"
  - ".NET cross-platform development"
- **OR** .NET 10.0 SDK: https://dotnet.microsoft.com/download/dotnet/10.0

### Cross-Platform Dependency Validation
For non-Windows environments (validation only):
- Install .NET 10.0: `curl -sSL https://dot.net/v1/dotnet-install.sh | bash /dev/stdin --version 10.0.100`
- Add to PATH: `export PATH="$HOME/.dotnet:$PATH"`

### Build Commands

#### Windows Full Build
- Restore dependencies: `dotnet restore Text-Grab.sln` -- takes 30 seconds initial, 2 seconds cached. NEVER CANCEL. Set timeout to 60+ seconds.
- Build main project: `dotnet build Text-Grab/Text-Grab.csproj -c Release` -- takes 45 seconds. NEVER CANCEL. Set timeout to 90+ seconds.
- Build packaged version: `dotnet build Text-Grab-Package/Text-Grab-Package.wapproj -c Release` -- takes 60 seconds. NEVER CANCEL. Set timeout to 120+ seconds.
- Run tests: `dotnet test Tests/Tests.csproj` -- takes 30 seconds. NEVER CANCEL. Set timeout to 60+ seconds.

#### Production Build (Windows Only)
- Use PowerShell script: `.\build-unpackaged.ps1` -- takes 180 seconds (3 minutes). NEVER CANCEL. Set timeout to 300+ seconds.
- Creates multiple architecture builds (x64, ARM64) with framework-dependent and self-contained variants

#### Cross-Platform Validation (Limited)
- **CRITICAL: Always use `-p:EnableWindowsTargeting=true` flag on non-Windows platforms**
- Restore solution: `dotnet restore Text-Grab.sln -p:EnableWindowsTargeting=true` -- takes 30 seconds initial, 2 seconds cached. NEVER CANCEL. Set timeout to 60+ seconds.
- Restore individual project: `dotnet restore Text-Grab/Text-Grab.csproj -p:EnableWindowsTargeting=true` -- takes 2-30 seconds. NEVER CANCEL. Set timeout to 60+ seconds.
- **DO NOT attempt full build** - will fail with Windows packaging error: `Microsoft.Build.Packaging.Pri.Tasks.ExpandPriContent task could not be loaded`

### Running the Application (Windows Only)
- Debug in Visual Studio: Set Text-Grab-Package as startup project, press F5
- Command line debug: `dotnet run --project Text-Grab/Text-Grab.csproj`
- Production executable: `Text-Grab/bin/Release/net10.0-windows10.0.22621.0/Text-Grab.exe`

### CLI Usage (Windows Only)
The application supports command-line arguments:
- `Text-Grab.exe Fullscreen` - Launch fullscreen grab mode
- `Text-Grab.exe GrabFrame` - Launch grab frame
- `Text-Grab.exe EditText` - Launch edit text window
- `Text-Grab.exe Settings` - Open settings
- `Text-Grab.exe "image.png"` - OCR an image file
- `Text-Grab.exe "C:\folder"` - OCR all images in folder

## Validation Scenarios

**ALWAYS run these validation steps after making changes (Windows only):**

### Core OCR Functionality
1. **Image OCR Test**: Run `dotnet test Tests/Tests.csproj` and verify OcrFontSampleImage test passes - validates basic OCR engine
2. **QR Code Reading**: Run `dotnet test Tests/Tests.csproj` and verify ReadQrCode test passes - validates barcode/QR functionality  
3. **Table Analysis**: Run `dotnet test Tests/Tests.csproj` and verify AnalyzeTable test passes - validates structured data extraction

### Manual Application Testing (Windows Only)
1. **Full-Screen Mode**: Launch app → Press Windows+Shift+T → Select screen region → Verify text copied to clipboard
2. **Grab Frame Mode**: Launch app → Create grab frame → Position over text → Click Grab → Verify OCR results
3. **Edit Text Window**: Launch app → Open Edit Text → Test text manipulation tools (find/replace, regex, etc.)
4. **Quick Lookup**: Press Windows+Shift+Q → Test custom text storage and retrieval

### Build Validation
- Always run `dotnet test Tests/Tests.csproj` before committing changes
- Use `.\build-unpackaged.ps1` to verify production builds work correctly
- Check CI/CD status: `.github/workflows/buildDev.yml` runs on Windows-latest only

## Key Project Structure

### Primary Components
- **Text-Grab/**: Main WPF application (.NET 10.0)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheJoeFin/Text-Grab](https://github.com/TheJoeFin/Text-Grab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
