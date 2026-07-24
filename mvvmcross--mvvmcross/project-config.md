---
trigger: always_on
description: MvvmCross is a cross-platform MVVM framework for .NET supporting Android, iOS, MacCatalyst, TvOS, macOS, WinUI, and WPF. This guide provides essential commands and workflows for GitHub Copilot to work effectively with the codebase.
---

# MvvmCross Development Instructions

MvvmCross is a cross-platform MVVM framework for .NET supporting Android, iOS, MacCatalyst, TvOS, macOS, WinUI, and WPF. This guide provides essential commands and workflows for GitHub Copilot to work effectively with the codebase.

**Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

## Platform Requirements

**CRITICAL PLATFORM LIMITATION:** All MvvmCross projects require iOS workloads even for Android and Core development due to multi-targeting. **Full builds and tests require Windows or macOS.**

### System Requirements

**Required**
- .NET 9.0.304 SDK or newer (used in CI/CD, see .github/actions/shared/action.yml)
- JDK 17 (required for Android development)
- Git with proper autocrlf configuration

**Windows (Full Development)**
- All workloads: `dotnet workload install android ios tvos macos maccatalyst maui-ios maui-android`
- Android SDK with platform-tools and SDK 35 (API Level 35)
- Use solution file: `MvvmCross.slnx`

**macOS (Full Development)** 
- All workloads: `dotnet workload install android ios tvos macos maccatalyst maui-ios maui-android`
- Android SDK with platform-tools and SDK 35 (API Level 35) 
- Xcode 16.4 (required for iOS development)
- Use solution filter: `MvvmCross-macos.slnf`

**Linux (Limited Development)**
- Android workload only: `dotnet workload install android`
- Android SDK with platform-tools and SDK 35 (API Level 35)
- Use solution filter: `MvvmCross-linux.slnf`
- Format checking and some tools work on individual projects

## Working Effectively

### Initial Setup Commands

Always run these commands after cloning:

```bash
# Install .NET 10.0.300 (used in CI/CD)
curl -sSL https://dot.net/v1/dotnet-install.sh | bash /dev/stdin --version 10.0.300

# Install JDK 21 (required for Android development)
# Use Eclipse Temurin distribution consistently across all platforms:
# Windows: Download from https://adoptium.net/temurin/releases/ or use winget install EclipseAdoptium.Temurin.21.JDK
# macOS: brew install --cask temurin@21
# Linux: Install via Adoptium APT repository or download from https://adoptium.net/temurin/releases/

# Restore .NET tools (includes SonarScanner, ReportGenerator, CycloneDX)
dotnet tool restore

# Install required workloads (Windows/macOS only - full command)
dotnet workload install android ios tvos macos maccatalyst maui-ios maui-android

# Linux limited workload
dotnet workload install android

# Configure git line endings
# Windows: git config --global core.autocrlf true  
# Mac/Linux: git config --global core.autocrlf input

# CRITICAL: Ensure full git history for version calculations
git fetch --unshallow
```

**Additional Requirements for Android Development:**
- Install Android SDK with platform-tools and SDK 35 (API Level 35)
- Android SDK can be installed via Android Studio or command line tools
- Set ANDROID_HOME environment variable to point to your Android SDK installation

**Additional Requirements for iOS Development (macOS only):**
- Install Xcode 16.4 from the Mac App Store or Apple Developer portal
- Accept Xcode license: `sudo xcodebuild -license accept`

### Build and Test Commands

**NEVER CANCEL builds or tests. Use 60+ minute timeouts.**

#### Full Build (Windows/macOS Only)

```bash
# Primary build command - takes 15-25 minutes. NEVER CANCEL. Set timeout to 60+ minutes.
dotnet run --project build/Build.csproj -- --verbosity=Minimal

# Build with specific configuration
dotnet run --project build/Build.csproj -- --configuration=Debug --verbosity=Minimal

# Build with artifacts output
dotnet run --project build/Build.csproj -- --artifactsDir=./output --ctrfDir=./ctrf
```

#### Unit Tests (Windows/macOS Only)

```bash  
# Run unit tests - takes 10-15 minutes. NEVER CANCEL. Set timeout to 30+ minutes.
dotnet run --project build/Build.csproj -- --target=UnitTest --verbosity=Minimal

# Tests generate reports in artifacts/Tests/ and ctrf/ directories
```

#### Available Build Targets

- `Clean` - Clean all build outputs
- `Restore` - Restore NuGet packages  
- `Build` - Build all projects
- `UnitTest` - Run all unit tests (depends on Build)
- `Sonar` - Full SonarCloud analysis pipeline
- `GenerateSBOM` - Generate Software Bill of Materials
- `CopyPackages` - Copy built packages to artifacts
- `Default` - Clean + Restore + Build + UnitTest

### Code Quality and Formatting

**Always run before committing changes:**

```bash
# Format code - REQUIRED before PR submission or CI fails
# Windows/macOS with full solution:
dotnet format whitespace --verify-no-changes MvvmCross.slnx

# Linux/Individual project approach (when solution fails):
dotnet format whitespace --verify-no-changes build/Build.csproj
dotnet format whitespace --verify-no-changes MvvmCross/MvvmCross.csproj

# Format specific plugin projects
dotnet format whitespace --verify-no-changes MvvmCross.Plugins/Color/MvvmCross.Plugin.Color.csproj

# Check formatting without changes
dotnet format whitespace --verify-no-changes --verbosity=diagnostic
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MvvmCross/MvvmCross](https://github.com/MvvmCross/MvvmCross) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
