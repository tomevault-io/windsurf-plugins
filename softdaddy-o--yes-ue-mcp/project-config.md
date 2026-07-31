---
trigger: always_on
description: **Native C++ MCP Plugin for Unreal Engine 5.6+**
---

# yes-ue-mcp - Claude Instructions

**Native C++ MCP Plugin for Unreal Engine 5.6+**

## Project Overview

This plugin implements the Model Context Protocol (MCP) over HTTP, allowing AI assistants (Claude Code, Cursor, Windsurf, etc.) to inspect, analyze, and **modify** Unreal Engine projects through a standardized JSON-RPC API.

## GitHub Repositories

| Remote | Repository URL | Purpose |
|--------|----------------|---------|
| `origin` | https://github.com/softdaddy-o/yes-ue-mcp-private.git | Development (private) |
| `public` | https://github.com/softdaddy-o/yes-ue-mcp.git | Release (public) |

## Workflow Rules

### Issue-Driven Development
- **Every task MUST have a GitHub issue** - Create an issue before starting any task
- **Close the issue** when the task is complete with a commit referencing it (e.g., `Closes #123`)
- Use conventional commit format: `feat:`, `fix:`, `docs:`, `refactor:`

### Git Workflow

**Private Repository (origin)**
- Commit frequently - whenever one issue/task is finished
- Each commit should reference its issue number
- Push to `origin` for development work

```bash
git add -A && git commit -m "feat: add new tool

Closes #123"
git push origin main
```

**Public Repository (public)**
- Push aggregated changes only when a new version is ready
- Use version tags (v1.0.0, v1.1.0, etc.)
- Filter out `.claude/` directory when pushing to public

```bash
# Push to public with .claude/ filtered out
git push public main --force-with-lease
git tag v1.0.0
git push public --tags
```

**Publishing to Public Repo**
When pushing to the public repository, use git filter to exclude private files:
```bash
# Create a filtered branch for public release
git checkout -b release-temp
git filter-branch --tree-filter 'rm -rf .claude' HEAD
git push public release-temp:main --force
git checkout main
git branch -D release-temp
```

## Testing Environment

**Primary Test Project:** Elpis (Action RPG - UE 5.7)
- **Project Path:** `F:\src3\Covenant\ElpisClient\`
- **Plugin Install Path:** `F:\src3\Covenant\ElpisClient\Plugins\yes-ue-mcp\`
- **Version Control:** Perforce (plugin excluded via `.p4ignore`)
- **MCP Endpoint:** `http://127.0.0.1:8080/mcp`
- **Status:** Primary test environment

**Secondary Test Project:** GameAnimationSample56 (UE 5.6)
- **Project Path:** `F:\src_ue5\GameAnimationSample56\`
- **Plugin Install Path:** `F:\src_ue5\GameAnimationSample56\Plugins\yes-ue-mcp\`
- **MCP Endpoint:** `http://127.0.0.1:8081/mcp`
- **Status:** Active test environment for UE 5.6

### Deploying to Test Projects

Use `copy_plugin.ps1` to safely copy the plugin to test projects:

```powershell
# Copy to both projects (default)
.\copy_plugin.ps1

# Copy to Elpis only
.\copy_plugin.ps1 -Target Elpis

# Copy to GameAnimationSample56 only
.\copy_plugin.ps1 -Target GameAnim
```

**Excludes:** `.git`, `.claude`, `Tests`, `.pytest_cache`, `.github`, `Docs`, test files

**Config Override:** Target projects can override settings (e.g., `ServerPort`) via their own `Config/DefaultYesUeMcp.ini`

## Module Structure

- **YesUeMcp** (Runtime) - Core MCP protocol layer
- **YesUeMcpEditor** (Editor) - HTTP server + tool implementations

## Coding Standards

- Follow Epic's C++ coding conventions
- Use `YESUEMCP_API` / `YESUEMCPEDITOR_API` for exported symbols
- All tools inherit from `UMcpToolBase`
- Register tools in `FYesUeMcpEditorModule::RegisterBuiltInTools()`

## Version Management

**Version is defined in TWO places (keep in sync):**
1. `YesUeMcp.uplugin` - `VersionName` field (read by UE plugin system)
2. `Source/YesUeMcp/Public/YesUeMcp.h` - `YESUEMCP_VERSION` macro (compile-time constant)

**IMPORTANT: Always increment version when modifying code!**

- Use semantic versioning: `MAJOR.MINOR.PATCH`
  - **MAJOR**: Breaking changes to existing tools or protocol
  - **MINOR**: New features (new tools, new parameters)
  - **PATCH**: Bug fixes, internal improvements, documentation

**When to increment:**
- Adding a new tool → MINOR
- Adding new parameter to existing tool → MINOR
- Changing tool input/output schema → MINOR (or MAJOR if breaking)
- Bug fixes → PATCH
- Internal refactoring → PATCH
- Any code change that affects behavior → PATCH minimum

## Key Files

- `Source/YesUeMcp/Public/Tools/McpToolBase.h` - Base class for all tools
- `Source/YesUeMcp/Public/Tools/McpToolRegistry.h` - Tool registration
- `Source/YesUeMcpEditor/Public/Server/McpServer.h` - HTTP server
- `Source/YesUeMcpEditor/Public/Subsystem/McpEditorSubsystem.h` - Lifecycle management
- `Source/YesUeMcpEditor/Public/Utils/McpAssetModifier.h` - Write operation utilities

## MCP Server

- **Protocol:** MCP 2025-03-26 (Streamable HTTP) with JSON-RPC 2.0
- **Transport:** HTTP (Streamable HTTP)
- **Endpoint:** `/mcp`
- **Port:** 8080 (configurable)
- **CORS:** Enabled for cross-origin requests

## Available Tools (28 total)

**Note:** Many tools support a `world` parameter: `"editor"` (default) or `"pie"` to target the Play-In-Editor world.

### Read Tools (11) - Consolidated in v1.6.0

#### Blueprint Tools
| Tool | Description |
|------|-------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [softdaddy-o/yes-ue-mcp](https://github.com/softdaddy-o/yes-ue-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
