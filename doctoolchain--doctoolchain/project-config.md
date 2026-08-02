---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**docToolchain** is an open-source documentation generation tool implementing the "docs-as-code" approach. It's built with Groovy/Java using Gradle and targets cross-platform deployment (Linux, macOS, Windows).

- **Current Version**: 3.4.2 (ng branch)
- **Primary Language**: Groovy with Java 17
- **Build System**: Gradle 8.1.1
- **Primary Interface**: `dtcw` wrapper script (NOT direct Gradle commands)

## Essential Commands

### Build and Test

```bash
# Build the project (expect ~11 test failures out of 53 - this is normal)
./gradlew clean build

# Run core tests only (should always pass - 66 tests, 3 skipped)
./gradlew core:test

# Run all tests (some failures expected if external tools missing)
./gradlew test

# Check for dependency updates
./gradlew dependencyUpdates
```

### Using dtcw Wrapper (Primary Interface)

**CRITICAL**: Always use `./dtcw` for docToolchain operations, not direct Gradle commands.

```bash
# Check version and status
./dtcw --version

# List all docToolchain tasks
./dtcw tasks --group doctoolchain

# Install Java 17 locally
./dtcw local install java

# Install docToolchain locally
./dtcw local install doctoolchain

# Generate documentation (basic functionality test)
./dtcw local generateHTML    # Output: build/html5/manual_test_script.html (~37KB)
./dtcw local generatePDF     # Output: build/pdf/manual_test_script.pdf (~62KB)

# Generate complete microsite
./dtcw local generateSite

# Preview microsite
./dtcw local previewSite
```

### Development Workflow

```bash
# 1. Validate shell scripts
find . -name "*.sh" -exec shellcheck {} \;

# 2. Run full CI pipeline locally
./.ci.sh

# 3. Build core module and create distribution
./gradlew core:jar
./gradlew prepareDist
./gradlew createDist
```

### Running Tests

```bash
# Unit tests (Spock framework in core module)
./gradlew core:test --info

# Integration tests (BATS - requires bats-core installed)
cd test && bats *.bats

# Test specific environment
bats test/local_environment.bats
bats test/docker_environment.bats
bats test/sdk_installation.bats
```

## Architecture and Code Organization

### Directory Structure

```
docToolchain/
├── core/                          # Core Java/Groovy library
│   ├── src/main/groovy/org/docToolchain/
│   │   ├── tasks/                 # Gradle task implementations
│   │   ├── atlassian/             # Jira & Confluence integration
│   │   │   ├── jira/              # Jira client, converters, utilities
│   │   │   └── confluence/        # Confluence publishing
│   │   ├── configuration/         # Configuration handling
│   │   └── scripts/               # Core script utilities
│   └── src/test/groovy/           # Spock unit tests
├── scripts/                       # Gradle task scripts (plugins)
│   ├── AsciiDocBasics.gradle      # Core document generation
│   ├── publishToConfluence.gradle # Confluence publishing
│   ├── exportJiraIssues.gradle    # Jira integration
│   ├── exportEA.gradle            # Enterprise Architect export
│   ├── generateSite.gradle        # Microsite generation (jBake)
│   └── [20+ other task scripts]
├── src/
│   ├── docs/                      # Documentation sources (AsciiDoc)
│   │   ├── 010_manual/            # User manual
│   │   ├── 015_tasks/             # Task documentation
│   │   ├── 020_tutorial/          # Tutorials
│   │   └── images/                # Image assets
│   ├── site/                      # Microsite templates (jBake)
│   └── test/                      # Test resources
├── test/                          # BATS integration tests
├── bin/                           # Shell scripts (doctoolchain, autobuildSite.bash)
├── template_config/               # Default configuration templates
├── dtcw, dtcw.ps1, dtcw.bat      # Cross-platform wrapper scripts
├── build.gradle                   # Main build configuration
├── docToolchainConfig.groovy      # Project configuration
├── Config.groovy                  # Alternative configuration file
└── libs.versions.toml             # Dependency version catalog
```

### Core Architecture

#### 1. Multi-Layer Build System
- **Wrapper Layer**: `dtcw` scripts abstract environment complexity (local/docker/sdk)
- **Gradle Layer**: Main build orchestration in `build.gradle`
- **Core Module**: Standalone library in `/core` with task implementations
- **Script Plugins**: Modular Gradle scripts in `/scripts` directory

#### 2. Task Organization
All tasks are implemented in `/scripts/*.gradle` files and applied in `build.gradle`:
- Each script file represents a feature/capability (e.g., `exportJiraIssues.gradle`)
- Tasks use core library classes from `org.docToolchain.tasks.*`
- Configuration-driven via `docToolchainConfig.groovy` or `Config.groovy`

#### 3. Core Package Structure
```
org.docToolchain/
├── tasks/                         # Task base classes
│   ├── DocToolchainTask.groovy    # Base task
│   ├── AbstractConfluenceTask     # Confluence operations
│   ├── ExportJiraIssuesTask       # Jira export
│   └── [other task implementations]
├── atlassian/                     # External integrations
│   ├── jira/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [docToolchain/docToolchain](https://github.com/docToolchain/docToolchain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
