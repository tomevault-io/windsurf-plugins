---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MacMCP is a Model Context Protocol (MCP) server that exposes macOS accessibility APIs to Large Language Models (LLMs) over the stdio protocol. It allows LLMs like Claude to interact with macOS applications using the same accessibility APIs available to users, enabling them to perform user-level tasks such as browsing the web, creating presentations, working with spreadsheets, or using messaging applications.


This  is the REAL APP. We never use mocks. not in the code, not in the tests
our tests have access to a live macos UI. we can interact with the ui in tests


## Build and Development Commands

### Building the Project
```bash
# Navigate to the MacMCP directory
cd MacMCP

# Build the project (debug build)
swift build

# Build the project (release build)
swift build -c release
```

### Running the Project
```bash
# Direct execution (for debugging)
./.build/debug/MacMCP

# Run with arguments
./.build/debug/MacMCP --debug

# Use the wrapper script for Claude desktop integration
../mcp-macos-wrapper.sh
```

### Code Formatting and Linting
```bash
# Format all Swift code and run linting
./scripts/format.sh

# Check formatting and linting without making changes (for CI)
./scripts/lint.sh
```

### Testing
```bash
# Run all tests (serialized execution)
cd MacMCP
swift test --no-parallel

# Run specific test
swift test --filter MacMCPTests.BasicArithmeticE2ETests/testAddition --no-parallel

# Run tests with verbose output
swift test --verbose --no-parallel

# Run tests with a specific number of workers (when parallel execution is needed for non-UI tests)
swift test --num-workers 1

# Run tests with code coverage
swift test --no-parallel --enable-code-coverage
```

### Accessibility Permissions
The MCP server automatically checks all required permissions at startup and displays a native macOS dialog to guide users through granting any missing permissions. No manual checking is required.

### Using the Accessibility Inspector Tools

#### Native Accessibility Inspector (Direct API Access)
```bash
# Navigate to the MacMCP directory
cd MacMCP

# Build the tool
swift build

# Run the tool to inspect an application by bundle ID
./.build/debug/ax-inspector --app-id com.apple.calculator

# Run the tool to inspect an application by process ID
./.build/debug/ax-inspector --pid 12345

# Filter output to show only specific UI component types
./.build/debug/ax-inspector --app-id com.apple.calculator --show-menus
./.build/debug/ax-inspector --app-id com.apple.calculator --show-window-controls
./.build/debug/ax-inspector --app-id com.apple.calculator --show-window-contents

# Hide invisible or disabled elements
./.build/debug/ax-inspector --app-id com.apple.calculator --hide-invisible --hide-disabled

# Save the output to a file
./.build/debug/ax-inspector --app-id com.apple.calculator --save output.txt

# Apply custom property filters
./.build/debug/ax-inspector --app-id com.apple.calculator --filter "role=button"
```

#### MCP-Based Accessibility Inspector (Uses MCP Tools) - PREFERRED METHOD
```bash
# Navigate to the MacMCP directory
cd MacMCP

# Build the tool
swift build

# Run the tool to inspect an application by bundle ID (shows what the MCP server "sees")
./.build/debug/mcp-ax-inspector --app-id com.apple.calculator --mcp-path ./.build/debug/MacMCP

# Filter output to show only buttons
./.build/debug/mcp-ax-inspector --app-id com.apple.calculator --mcp-path ./.build/debug/MacMCP --filter "role=AXButton"

# Find elements by description (useful for finding numeric buttons in Calculator)
./.build/debug/mcp-ax-inspector --app-id com.apple.calculator --mcp-path ./.build/debug/MacMCP --filter "description=1"

# Other filtering options work the same as the native inspector
./.build/debug/mcp-ax-inspector --app-id com.apple.calculator --mcp-path ./.build/debug/MacMCP --show-window-contents
./.build/debug/mcp-ax-inspector --app-id com.apple.calculator --mcp-path ./.build/debug/MacMCP --hide-invisible

# Save output to a file
./.build/debug/mcp-ax-inspector --app-id com.apple.calculator --mcp-path ./.build/debug/MacMCP --save output.txt

# Limit the tree depth for large applications
./.build/debug/mcp-ax-inspector --app-id com.apple.calculator --mcp-path ./.build/debug/MacMCP --max-depth 5

# NEW: Display detailed menu structure - shows all application menus
./.build/debug/mcp-ax-inspector --app-id com.apple.calculator --mcp-path ./.build/debug/MacMCP --menu-detail

# NEW: Get items for a specific menu
./.build/debug/mcp-ax-inspector --app-id com.apple.TextEdit --mcp-path ./.build/debug/MacMCP --menu-path "File"

# NEW: Get detailed window information
./.build/debug/mcp-ax-inspector --app-id com.apple.calculator --mcp-path ./.build/debug/MacMCP --window-detail

# NEW: Combine multiple options for comprehensive inspection
./.build/debug/mcp-ax-inspector --app-id com.apple.TextEdit --mcp-path ./.build/debug/MacMCP --menu-detail --window-detail --max-depth 3
```

#### Choosing Between the Inspectors

While both accessibility inspectors provide valuable insights, the **MCP-based inspector** is now the **preferred tool**:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/obra) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
