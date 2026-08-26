---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Rhombus.WinFormsMcp** is a Model Context Protocol (MCP) server that provides headless automation for WinForms applications using FlaUI with the UIA2 backend. The project structure follows the renamed pattern from `fnWindowsMCP` to `Rhombus.WinFormsMcp`.

## Build Commands

```bash
# Build the solution
dotnet build Rhombus.WinFormsMcp.sln

# Build for release
dotnet build Rhombus.WinFormsMcp.sln --configuration Release

# Restore dependencies
dotnet restore Rhombus.WinFormsMcp.sln

# Publish the server
dotnet publish src/Rhombus.WinFormsMcp.Server/Rhombus.WinFormsMcp.Server.csproj -c Release -o publish
```

## Test Commands

```bash
# Run all tests
dotnet test Rhombus.WinFormsMcp.sln

# Run tests with detailed output
dotnet test Rhombus.WinFormsMcp.sln --logger "console;verbosity=detailed"

# Run tests with code coverage
dotnet test Rhombus.WinFormsMcp.sln --collect:"XPlat Code Coverage"

# Run tests for release configuration
dotnet test Rhombus.WinFormsMcp.sln --configuration Release --no-build
```

## Run Commands

```bash
# Run the MCP server
dotnet run --project src/Rhombus.WinFormsMcp.Server/Rhombus.WinFormsMcp.Server.csproj

# Run the test WinForms application
dotnet run --project src/Rhombus.WinFormsMcp.TestApp/Rhombus.WinFormsMcp.TestApp.csproj
```

## Package Commands

```bash
# Create NuGet package (auto-generated on build)
dotnet build src/Rhombus.WinFormsMcp.Server/Rhombus.WinFormsMcp.Server.csproj -c Release

# Pack explicitly
dotnet pack src/Rhombus.WinFormsMcp.Server/Rhombus.WinFormsMcp.Server.csproj -c Release
```

## Architecture

### Core Components

1. **Rhombus.WinFormsMcp.Server** (src/Rhombus.WinFormsMcp.Server/)
   - `Program.cs`: MCP server implementation with JSON-RPC 2.0 over stdio transport. Contains 33 tool implementations and SessionManager for element caching.
   - `Automation/AutomationHelper.cs`: Core FlaUI wrapper with 40+ automation methods. Provides process management, element discovery, UI interaction, validation, window management, clipboard, and event capabilities.

2. **Rhombus.WinFormsMcp.TestApp** (src/Rhombus.WinFormsMcp.TestApp/)
   - Sample WinForms application with various controls for testing automation capabilities.

3. **Rhombus.WinFormsMcp.Tests** (tests/Rhombus.WinFormsMcp.Tests/)
   - NUnit test suite covering AutomationHelper functionality, process lifecycle, element operations, and resource cleanup.

### Key Technical Decisions

- **Framework**: .NET 8.0 Windows-specific (net8.0-windows) for WinForms compatibility
- **UI Automation**: FlaUI 4.0.0 with UIA2 backend for maximum WinForms compatibility without visual requirements
- **Testing**: NUnit 3.14.0 with Moq for mocking
- **Protocol**: MCP with stdio transport, single-line JSON-RPC 2.0 messages
- **Package Distribution**: NuGet (Rhombus.WinFormsMcp), NPM (@fnrhombus/winforms-mcp)

### Code Organization

**File-per-class rule:** One public class per file is strongly preferred. Multiple classes in a single file are only acceptable when:
- The classes are tightly coupled (e.g., a public class and its direct helper)
- The types are single-use (e.g., enums, markers, small value objects used nowhere else)

**Avoid monolithic files.** Keep files focused and navigable. If a file grows to contain multiple unrelated public classes or becomes difficult to navigate, it should be split.

### MCP Tools Available

The server implements 33 tools via JSON-RPC:
- Process Management: `launch_app`, `attach_to_process`, `close_app`, `get_process_status`
- Element Discovery: `find_element`, `find_elements`, `element_exists`, `wait_for_element`, `get_element_tree`
- UI Interaction: `click_element`, `type_text`, `set_value`, `drag_drop`, `send_keys`, `select_item`, `click_menu_item`, `toggle_element`
- Property & State: `get_property`, `wait_for_condition`, `get_focused_element`
- Data Controls: `scroll_element`, `get_table_data`, `set_table_cell`
- Window Management: `manage_window`, `list_windows`
- Events: `raise_event`, `listen_for_event`, `open_context_menu`
- Visual: `take_screenshot`, `render_form`
- Clipboard & Misc: `get_clipboard`, `set_clipboard`, `read_tooltip`

### Session Management

The server maintains session state across tool calls:
- Cached automation elements with generated IDs (elem_1, elem_2, etc.)
- Active AutomationHelper instance
- Process tracking with PIDs
- Per-process desktop routing (hidden vs default desktop)
- Native process handles for exit code access (CreateProcess-launched processes)

### Headless Mode (Hidden Desktop)

When `HEADLESS=true`, the server creates a hidden desktop via `CreateDesktop("McpAutomation")` within `WinSta0`. Key implementation details:

- **Process launch:** Uses `CreateProcess` P/Invoke with `STARTUPINFO.lpDesktop = "WinSta0\\McpAutomation"` (cannot use `Process.Start`)
- **Element discovery:** Requires `SetThreadDesktop(hDesktop)` before FlaUI/UIA calls; handled transparently by `OnProcessDesktop()`
- **Screenshots:** Uses `PrintWindow(hwnd, hdc, PW_RENDERFULLCONTENT)` — flag 2, not flag 0 (flag 0 returns blank on hidden desktops)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fnrhombus/winforms-mcp](https://github.com/fnrhombus/winforms-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
