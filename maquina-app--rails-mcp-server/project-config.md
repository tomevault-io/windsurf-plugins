---
trigger: always_on
description: This guide helps AI agents (Claude, GPT, etc.) use the Rails MCP Server effectively. It covers tool selection, common patterns, and troubleshooting.
---

# Rails MCP Server - AI Agent Guide

This guide helps AI agents (Claude, GPT, etc.) use the Rails MCP Server effectively. It covers tool selection, common patterns, and troubleshooting.

## Architecture Overview

Rails MCP Server uses a **progressive tool discovery** pattern to minimize context usage:

```
MCP Client (Claude, etc.)
    │
    ▼
┌─────────────────────────────────────────────┐
│  4 MCP-Registered Tools                     │
│  ┌─────────────┐  ┌─────────────────────┐   │
│  │switch_project│  │search_tools        │   │
│  └─────────────┘  └─────────────────────┘   │
│  ┌─────────────┐  ┌─────────────────────┐   │
│  │execute_tool │──▶│ 9 Internal Analyzers│   │
│  └─────────────┘  └─────────────────────┘   │
│  ┌─────────────┐                            │
│  │execute_ruby │                            │
│  └─────────────┘                            │
└─────────────────────────────────────────────┘
```

**Key concept:** Only 4 tools are registered with MCP. The 9 internal analyzers (`analyze_models`, `get_routes`, etc.) are discovered via `search_tools` and invoked via `execute_tool`.

---

## Quick Start

**Always start with these two steps:**

```
# 1. Switch to the project
railsMcpServer:switch_project project_name: "your-project-name"

# 2. Get project overview
railsMcpServer:execute_tool tool_name: "project_info"
```

**If unsure what tools are available:**

```
railsMcpServer:search_tools
railsMcpServer:search_tools category: "models"
railsMcpServer:search_tools query: "routes"
```

---

## Tool Selection Guide

### Reading Files

**Primary method** - Use `execute_ruby` with `read_file()`:

```
railsMcpServer:execute_ruby code: "puts read_file('config/routes.rb')"
railsMcpServer:execute_ruby code: "puts read_file('app/models/user.rb')"
railsMcpServer:execute_ruby code: "puts read_file('app/controllers/users_controller.rb')"
```

**Alternative** - Use `get_file` tool:

```
railsMcpServer:execute_tool tool_name: "get_file" params: { path: "config/routes.rb" }
```

> ⚠️ **Important:** Do NOT use Claude's built-in `view` tool for Rails project files. It cannot access the project directory. Always use Rails MCP tools.

---

### Finding Files

**Use `execute_ruby` with `Dir.glob()`:**

```
# Find all models
railsMcpServer:execute_ruby code: "puts Dir.glob('app/models/**/*.rb').join('\n')"

# Find all controllers
railsMcpServer:execute_ruby code: "puts Dir.glob('app/controllers/**/*.rb').join('\n')"

# Find files by name pattern
railsMcpServer:execute_ruby code: "puts Dir.glob('app/**/*user*').join('\n')"

# Find all view templates
railsMcpServer:execute_ruby code: "puts Dir.glob('app/views/**/*.erb').join('\n')"

# Find Stimulus controllers
railsMcpServer:execute_ruby code: "puts Dir.glob('app/javascript/controllers/**/*.js').join('\n')"
```

**Using `list_files` helper** (glob pattern):

```
# List Ruby files in models directory
railsMcpServer:execute_ruby code: "puts list_files('app/models/**/*.rb')"
```

---

### Analyzing Models

```
# List all models
railsMcpServer:execute_tool tool_name: "analyze_models"

# Analyze specific model with associations, validations, callbacks
railsMcpServer:execute_tool tool_name: "analyze_models" params: { model_name: "User" }

# Analyze multiple models
railsMcpServer:execute_tool tool_name: "analyze_models" params: { model_names: ["User", "Post", "Comment"] }

# Quick list (names only)
railsMcpServer:execute_tool tool_name: "analyze_models" params: { detail_level: "names" }

# With Prism static analysis (callbacks, scopes, methods)
railsMcpServer:execute_tool tool_name: "analyze_models" params: { model_name: "User", analysis_type: "full" }
```

---

### Getting Database Schema

```
# List all tables
railsMcpServer:execute_tool tool_name: "get_schema" params: { detail_level: "tables" }

# Get specific table schema
railsMcpServer:execute_tool tool_name: "get_schema" params: { table_name: "users" }

# Get multiple tables
railsMcpServer:execute_tool tool_name: "get_schema" params: { table_names: ["users", "posts"] }

# Full schema with indexes
railsMcpServer:execute_tool tool_name: "get_schema"
```

---

### Getting Routes

```
# All routes
railsMcpServer:execute_tool tool_name: "get_routes"

# Filter by controller
railsMcpServer:execute_tool tool_name: "get_routes" params: { controller: "users" }

# Filter by HTTP verb
railsMcpServer:execute_tool tool_name: "get_routes" params: { verb: "POST" }

# Filter by path
railsMcpServer:execute_tool tool_name: "get_routes" params: { path_contains: "api" }

# Named routes only
railsMcpServer:execute_tool tool_name: "get_routes" params: { named_only: true }
```

**Fallback if `get_routes` fails:**

```
railsMcpServer:execute_ruby code: "puts read_file('config/routes.rb')"
```

---

### Analyzing Controllers

```
# List all controllers
railsMcpServer:execute_tool tool_name: "analyze_controller_views" params: { detail_level: "names" }

# Analyze specific controller (actions, callbacks, views)
railsMcpServer:execute_tool tool_name: "analyze_controller_views" params: { controller_name: "users" }

# With Prism analysis (filters, strong params, instance variables)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maquina-app/rails-mcp-server](https://github.com/maquina-app/rails-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
