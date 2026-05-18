---
trigger: always_on
description: > Bridge R computational environments and Large Language Models
---

# btw

> Bridge R computational environments and Large Language Models

## Overview

btw is an R package that helps humans and LLMs work together with R by providing utilities to describe R objects, package documentation, and workspace state in LLM-friendly plain text. The package offers a flexible collection of tools that can be used interactively (copy-paste workflows), programmatically (direct function calls), or as enhanced chat clients (via ellmer or MCP servers).

The primary goal is creating a collection of tools useful to both LLMs and humans when working together with R, with an emphasis on flexibility of usage across different workflows and platforms.

## Quick Reference

- **Project type:** R Package
- **Language:** R (≥ 4.1.0)
- **Key frameworks:** ellmer (LLM chat integration), mcptools (Model Context Protocol), shiny and shinychat (chat app)

## Purpose and Design Philosophy

btw prioritizes flexibility of usage through multiple entry points:

- **`btw()`** - Interactive copy-paste workflow: gather context from R and paste into any chat interface
- **`btw_tools()`** - Register tools with ellmer chat clients for custom applications
- **`btw_client()` / `btw_app()`** - Batteries-included chat clients with your preferred LLM provider, model, and project context
- **MCP server** - Expose tools to third-party coding agents like Claude Desktop or Continue via `btw_mcp_server()`

Project configuration via `btw.md` files provides conversation stability across sessions by defining default provider, model, tools, and project-specific instructions. These files are treated as instructions for coding assistants and help avoid repeating context.

btw also serves as a laboratory for discovering best practices in LLM tool design - output formats and approaches evolve based on experimentation with what works best across different models.

## Key Design Decisions

- **Multiple entry points for flexibility** - Support interactive, programmatic, and agent-based workflows without forcing users into a single pattern
- **S3 dispatch via `btw_this()`** - Extensible backend for describing different R object types, with `btw_this.character()` handling command aliases for non-object concepts
- **Tool grouping for safety and clarity** - Tools organized into categories (docs, env, files, ide, search, session, web) to reduce LLM cognitive load and provide granular safety controls
- **Project configuration via `btw.md`** - Enables stable, repeatable conversations across sessions with project-specific context and preferred settings
- **S7 interoperability** - Uses S7 classes where required by ellmer's API while keeping most package code in familiar S3
- **Experimental format optimization** - Continuous testing of output formats (JSON for data, markdown for display, plain text for docs) to find what LLMs handle best

## How It Works

### Core Architecture

The package has three main layers:

1. **User-facing API** - `btw()` function accepts any R objects, help topics, file paths, or special commands (e.g., `"@platform_info"`, `"{dplyr}"`) and assembles them into LLM-ready context
2. **Description System** - `btw_this()` S3 generic dispatches to type-specific methods that know how to describe data frames, functions, environments, packages, etc.
3. **Tool System** - ~20 individual tools that can be registered with ellmer chats or MCP servers to give LLMs active capabilities beyond passive context
4. **Agents** - `btw_task_*()` functions provide guided workflows for common tasks and are primarily intended for interactive use, but can also be used as sub-agents via tool calls.

### Entry Points

**Interactive Copy-Paste (`btw()`)**
```r
btw(mtcars, "{dplyr}", ?dplyr::across)
# ✔ btw copied to the clipboard!
```
Assembles context and copies to clipboard for pasting into any chat interface.

**Programmatic Tool Registration (`btw_tools()`)**
```r
chat <- ellmer::chat_anthropic()
chat$register_tools(btw_tools("docs", "env"))  # Only doc and environment tools
```
Register specific tool groups with ellmer chat clients.

**Batteries-Included Chat (`btw_client()` / `btw_app()`)**
```r
chat <- btw_client(mtcars)  # Starts with mtcars in context
chat$chat("Summarize this data")

btw_app()  # Launches Shiny chat interface
```
Pre-configured chat clients with btw tools, project context from `btw.md`, and initial objects.

**MCP Server (`btw_mcp_server()`)**
```r
btw_mcp_server()  # Blocks; run non-interactively
```
Exposes btw tools to external MCP clients. Configure in Claude Desktop or other MCP-compatible tools.

### Tool System Design

Tools are defined in `R/tool-*.R` files following a consistent pattern:

1. **Implementation function** (`btw_tool_*_impl()`) - Does the actual work, returns a tool result object
2. **Exported wrapper** (`btw_tool_*()`) - Thin wrapper with roxygen docs for users
3. **Tool registration** - Called via `.btw_add_to_tools()` to register with ellmer

Tools are grouped by capability:
- **agent** - Hierarchical workflows via `btw_tool_agent_subagent()` to delegate tasks to specialized subagents
- **cran** - Search CRAN packages and retrieve package info
- **docs** - Package documentation, help pages, vignettes, NEWS
- **env** - Describe data frames and environments

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [posit-dev/btw](https://github.com/posit-dev/btw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
