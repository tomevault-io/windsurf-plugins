---
trigger: always_on
description: **ALWAYS follow these instructions first. Only fallback to additional search and context gathering if the information here is incomplete or found to be in error.**
---

# Copilot Instructions for openapi2zig

**ALWAYS follow these instructions first. Only fallback to additional search and context gathering if the information here is incomplete or found to be in error.**

## Project Overview

This is a CLI tool written in Zig that generates type-safe Zig API client code from OpenAPI/Swagger specifications. The architecture follows a unified converter pattern that normalizes both OpenAPI v3.0 and Swagger v2.0 specs into a common intermediate representation before code generation.

## Working Effectively

### Prerequisites and Installation
Install Zig 0.16.0 or newer:

**Option 1: GitHub Codespaces (RECOMMENDED for development)**
```bash
# Navigate to repository on GitHub → Code → Codespaces → Create codespace
# Everything is pre-configured with Zig 0.16.0, takes 2-3 minutes to set up
```

**Option 2: Dev Containers (local with Docker)**
```bash
# Install VS Code + Dev Containers extension
# Open project in VS Code → "Reopen in Container"
# Uses .devcontainer/devcontainer.json with Zig 0.16.0
```

**Option 3: Manual Installation**
```bash
# Download Zig 0.16.0 from https://ziglang.org/download/0.16.0/
# Linux x86_64:
wget https://ziglang.org/download/0.16.0/zig-linux-x86_64-0.16.0.tar.xz
tar -xf zig-linux-x86_64-0.16.0.tar.xz
export PATH="$PWD/zig-linux-x86_64-0.16.0:$PATH"

# Verify installation
zig version  # Must output "0.16.0" or newer
```

### Bootstrap, Build, and Test the Repository
Run these commands in order. NEVER CANCEL long-running operations:

```bash
# 1. Verify Zig installation
zig version

# 2. Build project (takes 2-5 minutes first time, NEVER CANCEL)
zig build -Doptimize=Debug  # Set timeout to 10+ minutes

# 3. Run comprehensive test suite (takes 1-3 minutes, NEVER CANCEL)
zig build test  # Set timeout to 10+ minutes

# 4. Test code generation with sample OpenAPI specs
zig build run-generate  # Generates both v2.0 and v3.0 samples, takes 30 seconds

# 5. Validate generated code compiles and runs
zig run generated/main.zig  # Should output "Generated models build and run !!"
```

### Validation Scenarios
ALWAYS run these validation steps after making any changes:

```bash
# Format check (REQUIRED before commits or CI fails)
zig fmt --check src/
zig fmt --check build.zig

# Build with all optimization levels (CI requirement)
zig build -Doptimize=Debug      # Development builds, 2-5 minutes
zig build -Doptimize=ReleaseFast  # Performance builds, 2-5 minutes  
zig build -Doptimize=ReleaseSafe  # Safe optimized builds, 2-5 minutes
zig build -Doptimize=ReleaseSmall # Size-optimized builds, 2-5 minutes

# Test the actual CLI functionality with sample specs
zig build run-generate-v3  # Generate from OpenAPI v3.0, 30 seconds
zig build run-generate-v2  # Generate from Swagger v2.0, 30 seconds
zig run generated/main.zig  # Test generated code, should output "Generated models build and run !!"

# Manual CLI testing with custom specs
./zig-out/bin/openapi2zig generate -i openapi/v3.0/petstore.json -o /tmp/test_output.zig
zig run /tmp/test_output.zig  # Verify custom generated code works

# Cross-compilation validation (1-2 minutes each)
zig build -Dtarget=x86_64-windows   # Windows x64
zig build -Dtarget=x86_64-macos     # macOS x64  
zig build -Dtarget=aarch64-linux    # Linux ARM64
```

## Key Architecture Patterns

### Multi-Version Support via Unified Document
- **Source specs**: Parsed into version-specific models (`src/models/v2.0/`, `src/models/v3.0/`)
- **Converters**: Transform version-specific models to unified representation (`src/generators/converters/`)
- **Unified generators**: Generate Zig code from unified document (`src/generators/unified/`)

Example flow: `Swagger 2.0 JSON → SwaggerDocument → SwaggerConverter → UnifiedDocument → UnifiedModelGenerator + UnifiedApiGenerator → Zig code`

### Memory Management Convention
All structs with dynamic allocations implement `deinit(allocator)` method. ALWAYS call `defer parsed.deinit(allocator)` after parsing operations. The `UnifiedDocument` owns all converted data and handles cleanup.

### CLI Pattern
- `src/cli.zig`: Argument parsing with structured `CliArgs` and `ParsedArgs` types
- `src/generator.zig`: Main orchestration - detects spec version, calls appropriate converter, generates output
- `src/detector.zig`: Version detection by parsing JSON for `openapi` or `swagger` fields

## Development Workflows

### Core Commands with Timeouts
```bash
# Development build with debug info (2-5 minutes, timeout: 10 minutes)
zig build -Doptimize=Debug

# Run comprehensive test suite (1-3 minutes, timeout: 10 minutes) 
zig build test

# Generate code from sample specs (30 seconds each, timeout: 2 minutes)
zig build run-generate-v3  # Uses openapi/v3.0/petstore.json
zig build run-generate-v2  # Uses openapi/v2.0/petstore.json
zig build run-generate     # Runs both

# Format check (5 seconds, REQUIRED before commits)
zig fmt --check src/
zig fmt --check build.zig

# Install test artifacts for debugging (30 seconds)
zig build install_test -Doptimize=Debug
```

### Version Information
The build system auto-generates `src/version_info.zig` from git tags/commits. Never edit this file manually.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [christianhelle/openapi2zig](https://github.com/christianhelle/openapi2zig) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
