---
trigger: always_on
description: This repository contains a collection of Claude Code plugins and agent skills for various development tasks. The main components include:
---

# awesome-agentskills Development Guide

## Project Overview

This repository contains a collection of Claude Code plugins and agent skills for various development tasks. The main components include:

- **tldraw-helper**: Plugin for creating diagrams and visualizations using tldraw Desktop
- **agent-skills-toolkit**: Tools for creating, testing, and optimizing agent skills
- **plugin-dev**: Plugin development utilities and templates

## MCP Servers

This project integrates the following MCP (Model Context Protocol) servers:

### Excalidraw MCP

**Configuration**: `.claude/mcp.json`

A remote MCP server for creating hand-drawn style diagrams with interactive editing capabilities.

- **URL**: `https://mcp.excalidraw.com/mcp`
- **Type**: HTTP (remote)
- **Features**:
  - Real-time hand-drawn diagram creation
  - Interactive fullscreen editing
  - Smooth viewport camera control
  - Supports architecture diagrams, flowcharts, and creative visualizations

**Configuration Format**:
```json
{
  "mcpServers": {
    "excalidraw": {
      "type": "http",
      "url": "https://mcp.excalidraw.com/mcp"
    }
  }
}
```

**Important Notes**:
- MCP servers **cannot** be configured in global `~/.claude/settings.json`
- Use project-level `.claude/mcp.json` for project-specific MCP servers
- Restart Claude Code after adding new MCP servers
- Check MCP server approval status with `/mcp` command

**Usage**: The Excalidraw MCP tools are automatically available in Claude Code after restart. Use them for creating casual, hand-drawn style diagrams.

**Comparison with tldraw-helper**:
- **Excalidraw**: Hand-drawn, casual style, browser-based interaction
- **tldraw-helper**: Professional, precise style, tldraw Desktop integration

## Development Workflow

### Important: Local Development First

**All plugin and skill development should be done in this project directory first**, not in the marketplace cache (`~/.claude/plugins/cache/`).

The marketplace cache at `~/.claude/plugins/cache/awesome-agent-skills/` is where Claude Code loads plugins from when they're installed. However, this is a **read-only copy** for runtime use. Any modifications made there will be:
- Lost when the plugin is updated
- Not tracked in version control
- Not shared with other developers

### Correct Development Process

1. **Make changes in the project directory:**
   ```
   /Users/likai/Github/Tools/awesome-agentskills/plugins/tldraw-helper/
   ```

2. **Test your changes locally** by symlinking or copying to the marketplace cache if needed

3. **Commit changes to git** in this project directory

4. **Publish updates** to the marketplace when ready

### Plugin Structure

Each plugin follows this structure:
```
plugins/plugin-name/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest
├── commands/                # Slash commands
├── skills/                  # Agent skills
├── agents/                  # Subagents
├── hooks/                   # Event hooks
├── README.md               # Plugin documentation
└── CHANGELOG.md            # Version history
```

## tldraw-helper Plugin

### Recent Improvements (2026-03-03)

**Critical Enhancement: Screenshot Verification**

Added mandatory screenshot verification step to prevent broken diagrams:

1. **Core Workflow** - Step 5 is now marked as "🚨 VERIFY VISUALLY (MANDATORY)"
2. **Detailed Checklist** - Added specific items to check in screenshots:
   - Text readability and overlap
   - Arrow positioning
   - Layout cleanliness
   - Element visibility
   - Professional appearance

3. **Common Mistakes** - "Not analyzing screenshots" is now the #1 most critical error

4. **Key Takeaways** - Emphasized that API success ≠ good diagram

**Why This Matters:**
The tldraw API returns success even if the diagram is completely unusable (overlapping text, arrows crossing labels, elements off-screen). Visual verification is the ONLY way to know if a diagram is correct.

### Testing Changes

When testing tldraw-helper changes:

1. Modify files in `plugins/tldraw-helper/`
2. Copy to marketplace cache if needed:
   ```bash
   cp -r plugins/tldraw-helper/* ~/.claude/plugins/cache/awesome-agent-skills/tldraw-helper/1.1.0/
   ```
3. Test with `/tldraw-helper:draw` command
4. Verify screenshot analysis is working correctly

## Git Workflow

### Current Status

```
M plugins/tldraw-helper/.claude-plugin/plugin.json
M plugins/tldraw-helper/README.md
M plugins/tldraw-helper/commands/draw.md
M plugins/tldraw-helper/skills/tldraw-canvas-api/SKILL.md
? plugins/tldraw-helper/CHANGELOG.md
? plugins/tldraw-helper/OPTIMIZATION_SUMMARY.md
? plugins/tldraw-helper/skills/tldraw-canvas-api/references/advanced-actions.md
```

### Committing Changes

When ready to commit:

```bash
# Stage specific files
git add plugins/tldraw-helper/skills/tldraw-canvas-api/SKILL.md
git add plugins/tldraw-helper/.claude-plugin/plugin.json
git add plugins/tldraw-helper/CHANGELOG.md

# Commit with descriptive message
git commit -m "Add mandatory screenshot verification to tldraw-helper skill

- Mark visual verification as mandatory step in workflow
- Add detailed checklist for screenshot analysis
- Promote 'not analyzing screenshots' to #1 critical error

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [libukai/awesome-agent-skills](https://github.com/libukai/awesome-agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
