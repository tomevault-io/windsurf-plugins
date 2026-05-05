---
trigger: always_on
description: Development guidance for Claude Code when working with this plugin marketplace.
---

# CLAUDE.md

Development guidance for Claude Code when working with this plugin marketplace.

## Quick Reference

```bash
# Validation
claude plugin validate .

# Local testing
/plugin marketplace add .
/plugin install <plugin>@wookstar-claude-plugins
/plugin marketplace update wookstar

# After changes
/plugin marketplace update wookstar
```

**Official Documentation:** <https://docs.claude.com/en/docs/claude-code/plugin-marketplaces.md>

## Architecture Decisions

### Consolidated Structure (v6.0)

All plugins live inside the `plugins/` directory for consistent organisation:

- **Standalone plugins** - focused plugins with skills, agents, commands, and optionally MCP servers
- **MCP-only plugins** - individual MCP server integrations (prefixed with `mcp-`)

**Why this approach:**

- All plugins in one location for easy discovery
- Each plugin has its own `.claude-plugin/plugin.json` manifest
- MCP servers declared in marketplace.json with file references
- Cross-platform compatible paths (forward slashes)

### Directory Structure

```
wookstar-claude-plugins/
├── .claude-plugin/
│   └── marketplace.json         # Root manifest (defines all plugins)
├── plugins/                     # ALL plugins live here
│   ├── claudecode/              # Commands-only plugin
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json
│   │   ├── README.md
│   │   └── commands/*.md
│   ├── developer/               # Plugin with MCP servers
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json
│   │   ├── .mcp.json            # MCP server configurations
│   │   ├── agents/*.md
│   │   ├── commands/*.md
│   │   └── skills/<name>/SKILL.md
│   ├── documents/
│   ├── message/                 # Rich text message drafts for Gmail, Outlook, and WhatsApp
│   ├── shopify-developer/
│   ├── ultimate-skill-creator/
│   ├── timezone-tools/          # Renamed from utilities
│   ├── google-apps-script/      # Extracted from productivity
│   ├── tampermonkey/            # Extracted from productivity
│   ├── git-worktrees/           # New (from productivity commands)
│   ├── google-tagmanager/       # Extracted from marketing
│   ├── google-analytics/        # Extracted from marketing
│   ├── google-ads-scripts/      # Extracted from marketing
│   ├── codex/                   # OpenAI Codex CLI headless mode
│   ├── gemini/                  # Gemini CLI headless mode
│   ├── ffmpeg/                  # FFmpeg CLI reference
│   ├── mcp-alphavantage/        # MCP-only plugins
│   ├── mcp-cloudflare/
│   ├── mcp-coingecko/
│   ├── mcp-currency-conversion/
│   ├── mcp-excalidraw/
│   ├── mcp-fetch/
│   ├── mcp-gemini-bridge/
│   ├── mcp-google-workspace/
│   ├── mcp-mikrotik/
│   ├── mcp-n8n/
│   ├── mcp-notion/
│   ├── mcp-open-meteo/
│   └── mcp-perplexity/
└── docs/                        # Development reference files
```

### Auto-Loading Rules

Claude Code auto-discovers components within each plugin:

| Directory | File Pattern | Loaded As |
|-----------|--------------|-----------|
| `agents/` | `*.md` | Agents |
| `commands/` | `*.md` | Commands |
| `skills/<name>/` | `SKILL.md` | Skills |
| Root | `.mcp.json` | MCP Servers |

## MCP Configuration Rules

**CRITICAL:** MCP servers use file references in marketplace.json, NOT in plugin.json.

```json
// In marketplace.json entry:
"mcpServers": "./.mcp.json"

// plugin.json does NOT need mcpServers field
```

**Why this pattern:**

1. Single source of truth - MCP config in one place
2. Marketplace controls which MCPs are loaded
3. plugin.json stays minimal (name, description, version, author)

**Validation checklist:**

- [ ] `.mcp.json` exists in plugin directory (for MCP plugins)
- [ ] marketplace.json uses `"mcpServers": "./.mcp.json"`
- [ ] plugin.json does NOT duplicate mcpServers
- [ ] `claude plugin validate .` passes

## Adding Components

### New Full-Featured Plugin

1. Create directory: `plugins/<plugin-name>/`
2. Create `.claude-plugin/plugin.json` with name, description, version, author
3. Create `README.md` for documentation
4. Add component directories: `agents/`, `commands/`, `skills/`
5. Add entry to `.claude-plugin/marketplace.json`
6. Test: `/plugin install <plugin>@wookstar-claude-plugins`

### New MCP-Only Plugin

1. Create directory: `plugins/mcp-<name>/`
2. Create `.claude-plugin/plugin.json` (without mcpServers)
3. Create `.mcp.json` with server configuration
4. Create `README.md` for documentation
5. Add entry to marketplace.json with `"mcpServers": "./.mcp.json"`
6. Test: `/plugin install mcp-<name>@wookstar-claude-plugins`

### Command/Agent (to existing plugin)

1. Create file: `plugins/<plugin>/commands/<name>.md` or `plugins/<plugin>/agents/<name>.md`
2. Auto-loaded on marketplace update
3. Test: `/plugin marketplace update wookstar`

### Skill (to existing plugin)

1. Create: `plugins/<plugin>/skills/<skill-name>/SKILL.md`
2. Optional subdirectories: `assets/`, `references/`, `scripts/`
3. Auto-loaded on marketplace update

### MCP Server (embedded in plugin)

1. Create/edit: `plugins/<plugin>/.mcp.json`
2. Add `"mcpServers": "./.mcp.json"` to marketplace.json entry
3. Test: `/plugin install <plugin>@wookstar-claude-plugins`

### LSP Plugin (intentional exception to "manifest required")


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [henkisdabro/wookstar-claude-plugins](https://github.com/henkisdabro/wookstar-claude-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
