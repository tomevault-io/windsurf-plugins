---
trigger: always_on
description: - Before making any changes or edits, talk through and mention your plan for the changes.
---

# Cursor Rules for Swift Package Projects
- Before making any changes or edits, talk through and mention your plan for the changes.

# Project Goals: MCP Template
This project aims to create a simple command line executable and Swift package to make creating MCPs (Model Control Protocol) for Mac very easy. The project will eventually support:
1. Command line stdio for direct MCP interaction
2. Command line stdio → standalone Mac app via Bonjour for networked MCP communication
3. SSE server in a Package → example command line app for SSE-based MCP

## Project Structure
- Package Name: mcp-template
- Swift Version: 6.0
- Platforms: macOS 15+, iOS 17+, macCatalyst 17+

### Package Targets:
1. EasyMCP (Library)
   - Purpose: Swift library for easy integration with the MCP protocol
   - Dependencies: mcp-swift-sdk (MCP module)
   - Key Files: 
     - Sources/EasyMCP/EasyMCP.swift - Main implementation that handles MCP server lifecycle, tool registration, and method handlers for listing and calling tools
     - Sources/EasyMCP/Logging.swift - Utility functions for structured logging with error, warning, and info levels

2. EasyMacMCP (Library)
   - Purpose: Mac-specific implementation for MCP with pipe communication supporting multiple concurrent MCP servers
   - Dependencies: EasyMCP
   - Key Files:
     - Sources/EasyMacMCP/EasyMacMCP.swift - Main generic implementation with type-safe Request/Response protocols
     - Sources/EasyMacMCP/HelperRequestPipe.swift - Actor that wraps WritePipe for sending requests from MCP servers to the Mac app
     - Sources/EasyMacMCP/HelperResponsePipe.swift - Actor that wraps ReadPipe for receiving responses from the Mac app
     - Sources/EasyMacMCP/MCPProtocols.swift - Defines protocols for MCP communication with tool metadata and conversion methods
     - Sources/EasyMacMCP/MCPTools.swift - Wraps MCP tools using ResponseManager for waiting for responses
     - Sources/EasyMacMCP/ResponseManager.swift - Actor that coordinates requests and responses, manages timeouts, and resolves async continuations
     - Sources/EasyMacMCP/ReadPipe.swift - Comprehensive implementation for creating, opening, and reading from named pipes with error handling
     - Sources/EasyMacMCP/WritePipe.swift - Counterpart to ReadPipe for creating, opening, and writing to named pipes with error handling
     - Sources/EasyMacMCP/PipeProtocols.swift - Defines protocol interfaces for PipeReadable and PipeWritable to allow mocking for testing
     - Sources/EasyMacMCP/PipeTestHelpers.swift - Utility functions to test pipe writing/reading with synchronous and asynchronous options
     - Sources/EasyMacMCP/FileManager+Pipe.swift - FileManager extension to check if a URL points to a named pipe using stat() system call
     - Sources/EasyMacMCP/Logging.swift - Mac-specific logging utilities with error, warning, and info levels that print to stderr/stdout

3. mcpexample (Executable)
   - Purpose: Command-line example using EasyMCP library
   - Dependencies: EasyMCP, ArgumentParser
   - Key Files:
     - Sources/mcpexample/MCPExample.swift - CLI implementation with ArgumentParser that demonstrates how to create and register MCP tools

4. EasyMCPTests (Test Target)
   - Purpose: Tests for the EasyMCP library
   - Key Files:
     - Tests/EasyMCPTests/MCPResponseReaderTests.swift - Tests for MCPResponseReader class
     - Tests/EasyMCPTests/ResponseManagerTests.swift - Tests for ResponseManager class

5. EasyMacMCPTests (Test Target)
   - Purpose: Tests for the EasyMacMCP library
   - Key Files:
     - Tests/EasyMacMCPTests/MCPToolsTests.swift - Tests for MCPTools that verify hello tools work correctly and handle timeouts
     - Tests/EasyMacMCPTests/MCPResponseReaderTests.swift - Tests for MCPResponseReader to ensure proper parsing and handling of responses
     - Tests/EasyMacMCPTests/ResponseManagerTests.swift - Tests for ResponseManager that verify response matching, timeout handling, and cancellation
     - Tests/EasyMacMCPTests/PipeConstants.swift - Constants for pipe paths used in testing
     - Tests/EasyMacMCPTests/MockPipe.swift - Mock implementations of PipeReadable and PipeWritable interfaces for unit testing

### MCPExample Targets:

The MCPExample targets demonstrate:
  1. How to use EasyMCP and EasyMacMCP to build a functional MCP server (via mcp-helper)
  2. How to use a running Mac app (MCPExample) as the "brains" that can connect to multiple MCP servers
  3. How to establish bidirectional communication between a GUI app and multiple command-line MCP servers

1. MCPExample (macOS App)
   - Purpose: macOS SwiftUI application that demonstrates MCP integration with multiple helpers
   - Dependencies: EasyMacMCP
   - Key Files:
     - MCPExample/MCPExample/MCPExampleApp.swift - Main app entry point
     - MCPExample/MCPExample/MCPMacApp.swift - Main implementation with handling of MCP server connections
     - MCPExample/MCPExample/ContentView.swift - SwiftUI interface with controls to test pipe communication and display messages
     - MCPExample/MCPExample/BuildSettings.swift - Shared build configuration including app group identifier for shared container access
     - MCPExample/MCPExample/HostRequestPipe.swift - Actor for reading requests from MCP helpers 

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adamwulf/mcp-template](https://github.com/adamwulf/mcp-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
