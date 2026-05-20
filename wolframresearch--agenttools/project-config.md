---
trigger: always_on
description: This file provides guidance to AI agents (Claude Code, GitHub Copilot, etc.) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI agents (Claude Code, GitHub Copilot, etc.) when working with code in this repository.

## Overview

AgentTools is a Wolfram Language package for integrating with AI agents and large language models. It provides MCP servers, agent skills, and other standard interfaces that enable AI systems to leverage Wolfram Language computation, Wolfram|Alpha knowledge, and related resources. The package supports a wide range of AI clients and protocols, with an extensible architecture for adding new tools, prompts, servers, and integration points.

## Development

Always use the WolframLanguageContext tool when working with Wolfram Language code to ensure that you are aware of the latest documentation and other Wolfram resources.

When you make changes to paclet source code, you should also write and run tests for the changes you made using the TestReport tool and check the updated files (including test files) with the CodeInspector tool.

If you need to debug code in the WolframLanguageEvaluator tool, you'll first need to evaluate:

```wl
PacletDirectoryLoad[ "path/to/AgentTools" ];
Get[ "Wolfram`AgentTools`" ]
```

Note: This is not necessary for the TestReport tool, since the tests load the paclet automatically.

You should use the SymbolDefinition tool to investigate symbols rather than use things like `DownValues`, `Definition`, etc. It runs in the same kernel as the WolframLanguageEvaluator tool, so it will have access to the same definitions.

## Writing and Running Tests

Use the TestReport MCP tool to run tests.

Always review [testing.md](docs/testing.md) for detailed instructions before modifying or adding tests.

## Building the Paclet

See [building.md](docs/building.md) for detailed instructions.

## Code Architecture

### Project Structure

- `Kernel/`: Contains the core implementation files
  - `AgentTools.wl`: Main entry point which loads an MX file if available, otherwise proceeds to `Main.wl`
  - `Main.wl`: Entry point for loading other package files; exported symbols must be declared here
  - `Common.wl`: Common utilities and [error handling](docs/error-handling.md)
  - `CommonSymbols.wl`: Any symbols shared between paclet files must be declared here
  - `CreateMCPServer.wl`: Implementation for creating MCP servers
  - `DefaultServers.wl`: Defines several predefined named MCP servers
  - `DeployAgentTools.wl`: Implementation for deploying and managing agent tool deployments
  - `Files.wl`: Helper functions for file operations
  - `Formatting.wl`: Definitions for formatting in notebooks
  - `InstallMCPServer.wl`: Implementation for installing MCP servers for use in some common MCP client applications
  - `MCPClientRequests.wl`: Server-to-client request infrastructure (request registry, response correlation, notification dispatch) used by [MCP roots](docs/mcp-roots.md) and other server-initiated requests
  - `MCPRoots.wl`: [MCP roots](docs/mcp-roots.md) handshake — issues `roots/list`, normalizes `file://` URIs (including malformed Windows variants), and applies the selected directory to the kernel, evaluator, and `RunProcess` calls
  - `MCPServerObject.wl`: Defines the MCP server object format
  - `Messages.wl`: Definitions for error messages
  - `PacletExtension.wl`: Paclet discovery, name resolution, and definition loading for the [paclet extension](docs/paclet-extensions.md) system
  - `PreferencesContent.wl`: Implementation of `CreatePreferencesContent`, which builds the toolset configuration UI for the system preferences dialog (see [preferences-content.md](docs/preferences-content.md))
  - `StartMCPServer.wl`: Implementation for starting MCP servers
  - `SupportedClients.wl`: Registry of supported MCP clients (`$SupportedMCPClients`) and relevant utility functions
  - `ValidateAgentToolsPacletExtension.wl`: Validation of `"AgentTools"` [paclet extensions](docs/paclet-extensions.md)
  - `UIResources.wl`: [MCP Apps](docs/mcp-apps.md) UI resource registry, client capability detection, and shared cloud notebook deployment helper
  - `Utilities.wl`: General-purpose helpers — LLMKit subscription checks, Chatbook version verification, and `toJSRegex` for converting ICU/PCRE patterns to ECMA 262 (used when sanitizing tool schema `"pattern"` fields)
  - `YAML.wl`: YAML import/export helpers (`importYAML`, `importYAMLString`, `exportYAML`, `exportYAMLString`) used by YAML-based MCP clients (e.g. Goose)
  - `Tools/`: Contains several files defining predefined MCP tools used by default servers. If tool schemas are modified, we need to rebuild agent skills.
  - `Prompts/`: Contains files defining predefined [MCP prompts](docs/mcp-prompts.md) used by default servers
- `Assets/`: Static assets bundled with the paclet
  - `Apps/`: HTML and JSON files for [MCP Apps](docs/mcp-apps.md) UI resources
- `FrontEnd/`: FrontEnd extension resources loaded by the notebook front end
  - `Assets/AgentTools.wl`: Localized strings (`AgentToolsStrings`) and graphics (`AgentToolsExpressions`) used by `CreatePreferencesContent` (see [preferences-content.md](docs/preferences-content.md))
- `Scripts/`: Contains utility scripts for building, testing, and running the paclet

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WolframResearch/AgentTools](https://github.com/WolframResearch/AgentTools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
