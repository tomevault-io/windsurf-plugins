---
trigger: always_on
description: Logi Plugin SDK plugin for adaptive ring functionality on Logitech devices with MCP (Model Context Protocol) integration.
---

# Logitum Adaptive Ring Plugin

Logi Plugin SDK plugin for adaptive ring functionality on Logitech devices with MCP (Model Context Protocol) integration.

## Features

- **Multi-Registry MCP Support**: Queries 3 major MCP registries (ToolSDK, Official, Glama) with 4500+ total servers
- **Smart Server Selection**: Intelligent ranking system selects the most general/relevant MCP server when multiple matches exist
- **Smart Caching**: Local SQLite database caches registry queries for instant lookups
- **Process Monitoring**: Detects app switches and automatically queries for relevant MCP servers
- **Actions Ring Management**: Dynamically updates Actions Ring with app-specific actions (keybinds, MCP prompts, Python scripts)
- **AI-Powered Suggestions**: Gemini AI suggests relevant workflows and actions based on app context and available MCP servers
- **MCP Tool Execution**: Executes MCP prompts, keybinds, and Python scripts through the Actions Ring
- **Usage Pattern Learning**: 5-layer intelligent system tracks UI interactions, processes workflows, clusters patterns, and ranks actions by frequency
- **Vector Clustering**: DBSCAN clustering of semantic workflows using VoyageAI embeddings for pattern discovery
- **Action Ranking**: Composite scoring algorithm (frequency + recency) automatically reorders actions by usage
- **Massive Coverage**: Supports developer tools, enterprise apps, databases, cloud platforms, and more
- **Zero Build Warnings**: Fully nullable reference type compliant

## Structure

- `AdaptiveRingPlugin/` - Main plugin implementation
  - `src/Models/` - Data models (MCP servers, actions, workflows, UI interactions)
  - `src/Services/` - Core services (Database, Registry Client, Process Monitor, Actions Ring Manager, AI services)
  - `src/Actions/` - Action implementations (AdaptiveRingCommand, CounterCommand, etc.)
  - `src/Helpers/` - Utility helpers (logging, resources, sanitization)
  - `src/Scripts/` - Python intelligence service for Gemini AI integration
- `DbCleaner/` - Utility for database maintenance

## Development

Build (run from `AdaptiveRingPlugin/src/` directory to avoid duplicate assembly errors):
```bash
cd AdaptiveRingPlugin/src
dotnet build
```

Or specify the full path:
```bash
dotnet build AdaptiveRingPlugin/src/AdaptiveRingPlugin.csproj
```

Clean:
```bash
cd AdaptiveRingPlugin/src
dotnet clean
```

## Implementation Details

### MCP Registry Integration

The plugin queries 3 MCP registries in cascading order:

1. **ToolSDK** (4109+ servers) - Local index downloaded on first run
   - API: `https://toolsdk-ai.github.io/toolsdk-mcp-registry/indexes/packages-list.json`
   - Cached locally for 7 days
   - Returns up to 10 matches for ranking

2. **Official MCP Registry** (380+ servers)
   - API: `https://registry.modelcontextprotocol.io/v0/servers`
   - Real-time search API
   - Collects all matches across search variants

3. **Glama** (aggregated sources)
   - API: `https://glama.ai/api/mcp/v1/servers`
   - Community-driven registry
   - Aggregates matches from multiple search terms

### Smart Server Selection

When multiple MCP servers match a query, the system ranks them by generality:

**Scoring Algorithm:**
- Exact match: +1000 points (e.g., "vscode" matches "vscode")
- Starts with search term: +700 points (e.g., "vscode-extension")
- Ends with search term: +600 points (e.g., "microsoft-vscode")
- Contains search term: +300 points
- Validated package: +200 points
- Feature-specific keywords: -200 points per keyword (api, extension, plugin, manager, etc.)
- Extra words after search term: -50 points per word
- Length penalty: -2 points per character over 8
- Special characters: -10 points each (-, _, .)
- Version suffixes: -50 points

**Example:** For "chrome", the system prefers "chrome" over "chrome-google-search-api" or "puppeteer-chrome-extension"

### Database Schema

SQLite database at: `C:\Users\panonit\AppData\Local\Logitum\adaptive_ring.db`

**Core Tables:**
- `remembered_apps` - Tracked applications
- `mcp_cache` - Caches MCP lookup results (7-day TTL)
- `toolsdk_index` - Local copy of ToolSDK registry
- `app_actions` - Actions for each app (keybinds, prompts, Python scripts) with usage tracking
- `ui_interactions` - Raw UI interaction events (24-hour TTL)
- `semantic_workflows` - AI-processed workflow descriptions
- `workflow_embeddings` - Vector embeddings for workflows (VoyageAI)
- `workflow_clusters` - Clustered workflow patterns with frequency counts

### Key Files

**Core Plugin:**
- `AdaptiveRingPlugin.cs` - Main plugin logic with service orchestration

**Services:**
- `Services/ProcessMonitor.cs` - Detects app switches using Win32 API
- `Services/AppDatabase.cs` - SQLite database with caching, indexing, and migrations
- `Services/MCPRegistryClient.cs` - Multi-registry query client with smart ranking
- `Services/ActionsRingManager.cs` - Manages Actions Ring UI and updates
- `Services/ActionPersistenceService.cs` - Persists and tracks action usage
- `Services/GeminiActionSuggestor.cs` - AI-powered action suggestions via Python service
- `Services/MCPPromptExecutor.cs` - Executes MCP prompts and manages connections
- `Services/MCPClient.cs` - MCP protocol client implementation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mrsladoje/logitum](https://github.com/mrsladoje/logitum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
