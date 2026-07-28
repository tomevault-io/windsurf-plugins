---
trigger: always_on
description: **ALWAYS reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**
---

# .NET IoT — GitHub Copilot Coding Agent Instructions

**ALWAYS reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

This repository contains the .NET IoT libraries: **System.Device.Gpio** (low-level GPIO APIs) and **Iot.Device.Bindings** (130+ device drivers/bindings for sensors, displays, motors, etc.). Code runs across different operating systems and hardware platforms.

## Working Effectively with Fresh Clone

### Required Dependencies and Setup
- **CRITICAL**: This repository requires **Azure DevOps feeds access** to build. If you get SDK resolution errors, this is due to network restrictions.
- **Install .NET 8.0.300 SDK**: Use `./dotnet.sh --version` to verify the exact version from global.json is available
- **Build command**: `./build.sh --restore --build` (Linux/macOS) or `Build.cmd` (Windows)
- **NEVER CANCEL**: Full repository build takes **30-45 minutes**. Set timeout to **60+ minutes**.

### Bootstrap and Build Process
Run these commands in order:
```bash
# 1. Ensure correct .NET SDK version (may download SDK automatically)
./dotnet.sh --version

# 2. Full repository build - NEVER CANCEL, takes 30-45 minutes
./build.sh --restore --build --configuration Release

# 3. Build with packages (for testing distribution)
./build.sh --restore --build --pack --configuration Release

# 4. Run tests - NEVER CANCEL, takes 15-30 minutes  
./build.sh --test --configuration Release
```

**If Azure DevOps feeds are inaccessible**: Build will fail with "Could not resolve SDK Microsoft.DotNet.Arcade.Sdk" errors. Document this as a known limitation requiring network access to Azure DevOps.

### Alternative Build Approaches
When main build fails due to network restrictions:
```bash
# Build individual samples (these use public NuGet packages)
cd samples/led-blink
../../dotnet.sh build

# Test basic .NET IoT functionality
./dotnet.sh new console -o test-project
cd test-project  
./dotnet.sh add package System.Device.Gpio
./dotnet.sh add package Iot.Device.Bindings
./dotnet.sh build && ./dotnet.sh run
```

## Repository Structure and Navigation

### Key Directories
- **`src/System.Device.Gpio/`** - Core GPIO library (pins, controllers, drivers)
- **`src/devices/`** - 130+ device-specific bindings (sensors, displays, motors)
- **`samples/`** - Standalone example projects demonstrating usage
- **`Documentation/`** - Comprehensive docs, contribution guidelines, tutorials
- **`tools/`** - Build tools, Arduino C# compiler, code generators

### Important Files
- **`global.json`** - Specifies required .NET SDK version (8.0.300)
- **`build.sh`/`Build.cmd`** - Main build scripts using Microsoft Arcade SDK
- **`azure-pipelines.yml`** - Official CI/CD pipeline configuration
- **`Directory.Build.props`** - Shared MSBuild properties and package references

### Finding Device Bindings
```bash
# List all 130+ available device bindings
ls src/devices/

# Find specific device (e.g., temperature sensors)
find src/devices -name "*temp*" -o -name "*dht*" -o -name "*bmp*"

# Each device has: library code, samples/, tests/, README.md
ls src/devices/Dhtxx/  # Example: DHT temperature sensor
```

## Testing and Validation

### Running Tests
```bash
# Run all tests - NEVER CANCEL, takes 15-30 minutes
./build.sh --test --configuration Release

# Run specific test project
./dotnet.sh test src/devices/Dhtxx/tests/

# Test count: 31 test projects across the repository
find . -name "*Test*.csproj" | wc -l
```

### Manual Validation Scenarios
**ALWAYS run these validation steps after making changes:**

1. **Build validation**: Ensure your changes don't break the overall build
2. **Sample execution**: Run relevant sample projects to verify functionality
3. **Hardware-independent testing**: Focus on parsing, calculation, and protocol logic
4. **Cross-platform considerations**: Verify code works on Linux, Windows, and embedded systems

### Validation for Device Bindings
```bash
# After modifying a device binding (e.g., Dhtxx):
cd src/devices/Dhtxx
../../../dotnet.sh build

# Test the sample
cd samples
../../../../dotnet.sh build
# Note: Hardware may not be available, verify compilation and logic

# Check for hardware abstraction compliance
grep -r "Environment.OSVersion\|DllImport\|P/Invoke" . # Should be minimal/none
```

## Common Commands and Expected Timing

| Command | Expected Time | Timeout Setting |
|---------|---------------|-----------------|
| `./build.sh --restore --build` | 30-45 minutes | 60+ minutes |
| `./build.sh --test` | 15-30 minutes | 45+ minutes |
| Individual device build | 1-3 minutes | 10 minutes |
| Sample project build | 30-60 seconds | 5 minutes |
| Full CI pipeline | 45-60 minutes | 90+ minutes |

### Linting and Code Quality
```bash
# Run markdown linting (used in CI)
npm install -g markdownlint-cli
markdownlint -c .markdownlint.json .

# Code follows .editorconfig and built-in analyzers
# Warnings/errors will appear during build - fix them
```

## Debugging Build Issues

### Common Issues and Solutions
1. **SDK resolution errors**: Azure DevOps feeds inaccessible - document as network limitation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dotnet/iot](https://github.com/dotnet/iot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
