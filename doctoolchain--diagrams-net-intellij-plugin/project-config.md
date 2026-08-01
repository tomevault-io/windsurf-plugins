---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an IntelliJ Platform plugin that integrates diagrams.net (formerly draw.io) directly into JetBrains IDEs. It provides a JCEF (Java Chromium Embedded Framework) based editor for diagram files with extensions `.drawio.(svg|png|xml)` and `.dio.(svg|png|xml)`, and auto-detects editable PNGs and SVGs created with diagrams.net.

The plugin uses an offline version of diagrams.net by default, so it works without an internet connection and keeps content local.

## Development Setup

### Prerequisites

This project uses SDKMAN for Java version management. **The required JDK is JDK 21.0.8-tem** as specified in `.sdkmanrc`:

```bash
# Install and use the correct JDK (21.0.8-tem)
sdk env install
```

**IMPORTANT**: All gradle commands MUST be run with the correct JDK activated:
```bash
source ~/.sdkman/bin/sdkman-init.sh && sdk env && ./gradlew <task>
```

The project requires exactly **JDK 21.0.8-tem (Temurin)**. Using a different JDK version will cause build failures.

### Submodule Initialization

The project includes diagrams.net as a git submodule at `src/webview/drawio`. **This must be initialized before building**:

```bash
git submodule update --init
```

The build will fail with an error if the submodule is not initialized.

## Common Development Commands

### Running and Testing

```bash
# Run IDE with plugin installed (for development/debugging)
./gradlew runIde

# Build the plugin
./gradlew build

# Run tests
./gradlew test

# Run all checks (verification)
./gradlew check
```

### Plugin-Specific Tasks

```bash
# Build plugin ZIP for distribution
./gradlew buildPlugin

# Verify plugin compatibility with target IDE versions
./gradlew verifyPlugin

# Patch plugin.xml with version info
./gradlew patchPluginXml

# Prepare sandbox environment
./gradlew prepareSandbox

# Sign plugin (requires credentials)
./gradlew signPlugin

# Publish to JetBrains Marketplace (requires PUBLISH_TOKEN env var)
./gradlew publishPlugin
```

### Testing with External IDEs

The `runTestIde` task runs an external IDE installation with fully isolated test configuration. This allows testing the plugin in different JetBrains IDEs (PyCharm, WebStorm, GoLand, etc.) without interfering with your default IDE installation.

```bash
# Run with default IntelliJ IDEA (checks ~/Applications and /Applications)
./gradlew runTestIde

# Run with a specific IDE
./gradlew runTestIde -PtestIdePath="$HOME/Applications/PyCharm.app"
./gradlew runTestIde -PtestIdePath="$HOME/Applications/WebStorm.app"
./gradlew runTestIde -PtestIdePath="$HOME/Applications/GoLand.app"

# Clean config before running (removes cached state)
./gradlew runTestIde -PtestIdePath="$HOME/Applications/PyCharm.app" -PcleanConfig=true

# Synchronous mode (for debugging, keeps Gradle running)
./gradlew runTestIde -PtestIdePath="$HOME/Applications/PyCharm.app" -Pdetached=false
```

**Key features:**
- **Full isolation**: Config, system, plugins, and log directories are separate from your default IDE
- **Per-IDE directories**: Each IDE type gets its own config directory (`build/test-ide-idea`, `build/test-ide-pycharm`, etc.)
- **Parallel runs**: You can run multiple IDEs simultaneously (default: detached mode)
- **Auto-detection**: Detects IDE type and uses correct environment variable prefix (IDEA, PYCHARM, WEBIDE, etc.)
- **JCEF workaround**: Automatically applies `-Dide.browser.jcef.out-of-process.enabled=false` to fix blank canvas issue

## Architecture

### Core Components

The plugin architecture consists of these key layers:

1. **Editor Provider Layer** (`DiagramsEditorProvider.kt`)
   - Entry point for IntelliJ's editor system
   - Determines if a file should be opened with the diagrams.net editor via `DiagramsFileUtil.isDiagramsFile()`
   - Creates `DiagramsEditor` instances
   - Contains workaround for JCEF cache lock issues (IJPL-148653)

2. **Editor Layer** (`DiagramsEditor.kt`)
   - Main `FileEditor` implementation
   - Manages the lifecycle of the `DiagramsWebView`
   - Handles file I/O operations (loading and saving)
   - Supports different file formats:
     - XML files (`.drawio`, `.drawio.xml`, `.dio`, `.dio.xml`) - saves as XML
     - SVG files (`.drawio.svg`, `.dio.svg`) - exports as SVG with XML header
     - PNG files (`.drawio.png`, `.dio.png`) - exports as PNG with embedded diagram data
   - Listens to theme changes and settings changes to reload the webview

3. **WebView Layer** (`BaseDiagramsWebView.kt`, `DiagramsWebView.kt`)
   - `BaseDiagramsWebView` manages JCEF browser instance and message passing
   - Registers custom `https://drawio-plugin` scheme handler to load offline diagrams.net assets
   - Implements bidirectional communication between Kotlin and JavaScript via `JBCefJSQuery`
   - `DiagramsWebView` handles diagram-specific operations (load, save, export)
   - Uses Jackson for JSON serialization/deserialization

4. **Message Protocol** (`DrawioWebMessages.kt`)
   - Defines sealed classes for bidirectional communication:
     - `OutgoingMessage.Request` (Export with response)
     - `OutgoingMessage.Event` (Load, Configure, Merge)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [docToolchain/diagrams.net-intellij-plugin](https://github.com/docToolchain/diagrams.net-intellij-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
