---
trigger: always_on
description: The Editor Guidelines repository contains a Visual Studio extension that adds vertical column guides to the text editor. It is written in C# targeting .NET Framework 4.7.2 and uses the Visual Studio SDK.
---

# Editor Guidelines - Visual Studio Extension

The Editor Guidelines repository contains a Visual Studio extension that adds vertical column guides to the text editor. It is written in C# targeting .NET Framework 4.7.2 and uses the Visual Studio SDK.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### CRITICAL: Platform Requirements
- **This project ONLY builds on Windows** - Do not attempt to build on Linux/macOS as it requires Visual Studio SDK and Windows-specific dependencies
- Requires Windows 10/11 with Visual Studio 2019 or 2022 installed
- Requires Visual Studio Workload: "Visual Studio extension development"
- Package source https://myget.org/F/vs-editor/api/v3/index.json is required but often blocked in sandboxed environments

### Build and Test Commands
**NEVER CANCEL any build or test commands - builds can take 10+ minutes**

#### Prerequisites Setup
```bash
# Ensure .NET Core 8.0 is installed (required by CI)
dotnet --version  # Should be 8.0.x

# On Windows only - verify Visual Studio and MSBuild are available
where msbuild  # Should find MSBuild.exe
```

#### Build Process (Windows Only)
```bash
# Navigate to repository root
cd /path/to/EditorGuidelines

# Restore and build the solution - NEVER CANCEL, takes 5-10 minutes
msbuild src /t:Restore,Build /p:Configuration=Debug /p:DeployExtension=false
# OR for Release builds:
msbuild src /t:Restore,Build /p:Configuration=Release /p:DeployExtension=false
```
**TIMEOUT: Set timeout to 15+ minutes. Build process includes package restoration and compilation of multiple projects.**

#### Running Tests (Windows Only)
```bash
# Run unit tests - NEVER CANCEL, takes 2-5 minutes  
dotnet test --no-build test\ColumnGuideTests\ColumnGuideTests.csproj --configuration Debug

# Run integration tests (if available)
dotnet test --no-build test\EditorGuidelinesTests\EditorGuidelinesTests.csproj --configuration Debug
```
**TIMEOUT: Set timeout to 10+ minutes for complete test suite.**

### Linux/Non-Windows Environment Limitations
```bash
# DO NOT attempt these commands on Linux - they will fail:
# msbuild src /t:Restore,Build  # ❌ Will fail - Windows only
# dotnet restore src/           # ❌ Will fail - MyGet source blocked
# dotnet build src/             # ❌ Will fail - Visual Studio SDK required

# Instead, document limitations:
echo "Build requires Windows environment with Visual Studio SDK"
echo "Package source https://myget.org/F/vs-editor/api/v3/index.json is often blocked"
```

## Key Project Structure

### Solution Layout
```
src/
├── Editor Guidelines.sln        # Main solution file
├── ColumnGuide/                 # Shared project with core functionality
│   ├── ColumnGuide.shproj      # Shared project file
│   ├── ColumnGuide.projitems   # Shared items definition
│   └── *.cs                    # Core extension code (~1900 lines)
├── VSIX/                       # VS 2015/2017/2019 extension package
│   └── VSIX.csproj            # Targets net472
└── VSIX_Dev17/                 # VS 2022 extension package  
    └── VSIX_Dev17.csproj      # Targets net472

test/
├── ColumnGuideTests/           # Unit tests using xUnit
│   └── ColumnGuideTests.csproj
└── EditorGuidelinesTests/      # Integration tests
    └── EditorGuidelinesTests.csproj
```

### Important Files for Development
- `src/ColumnGuide/EditorGuidelinesPackage.cs` - Main package class (288 lines)
- `src/ColumnGuide/ColumnGuideAdornment.cs` - Visual rendering logic (372 lines)  
- `src/ColumnGuide/Parser.cs` - .editorconfig parsing (251 lines)
- `src/ColumnGuide/TextEditorGuidesSettings.cs` - Settings management (306 lines)
- `Directory.Build.props` - Global build properties
- `.github/workflows/CI.yml` - Build automation

## Validation and Testing

### Manual Testing (Windows Only)
After making code changes, always validate by:

1. **Build the extension**: Use MSBuild commands above
2. **Install for testing**: 
   ```bash
   # Extension will be in src/VSIX/bin/Debug/ or src/VSIX_Dev17/bin/Debug/
   # Double-click the .vsix file to install in Visual Studio
   ```
3. **Test functionality**: Open Visual Studio, create a file, right-click editor → Guidelines menu
4. **Test .editorconfig support**: Create .editorconfig with `guidelines = 80,120` and verify lines appear

### Required Validation Steps
- Always build both Debug and Release configurations
- Test both VSIX projects (VS 2015-2019 and VS 2022 versions)
- Run complete unit test suite before submitting changes
- Verify extension loads correctly in Visual Studio experimental instance

### Pre-commit Validation
```bash
# These commands must pass before committing (Windows only):
msbuild src /t:Restore,Build /p:Configuration=Debug /p:DeployExtension=false
msbuild src /t:Restore,Build /p:Configuration=Release /p:DeployExtension=false
dotnet test --no-build test\ColumnGuideTests\ColumnGuideTests.csproj --configuration Debug
```

## Common Development Tasks

### Adding New Features
1. Modify shared code in `src/ColumnGuide/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pharring/EditorGuidelines](https://github.com/pharring/EditorGuidelines) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
