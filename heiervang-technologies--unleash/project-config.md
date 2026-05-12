---
trigger: always_on
description: This file provides context and instructions for AI agents working on the Unleash repository.
---

# Agent Instructions for Unleash

This file provides context and instructions for AI agents working on the Unleash repository.

## Self-Restart Capability

**IMPORTANT**: If you are running under the `unleash` wrapper, you can restart yourself to reload MCP servers, apply configuration changes, or recover from issues.

### How to Check if You Can Restart

Check the environment variable:
```bash
echo $AGENT_UNLEASH
```
If it returns `1`, you are running under the wrapper and can restart.

### How to Restart Yourself

Run this command via Bash:
```bash
unleash-refresh
```

Or with a custom message to receive after restart:
```bash
unleash-refresh "Continue working on the feature"
```

> **Note:** The old aliases `restart-claude` and `exit-claude` have been removed. Use `unleash-refresh` and `unleash-exit`.

### What Happens When You Restart

1. Your session is preserved (`--continue` flag added automatically)
2. You receive the message "RESURRECTED." (or your custom message)
3. MCP servers are reloaded with current configuration
4. You can continue where you left off

### When to Restart

- After MCP configuration changes (`.mcp.json` modified)
- When MCP servers become unresponsive
- To apply new plugin settings
- When instructed by the user

### Files Reference

| File | Purpose |
|------|---------|
| `scripts/unleash-refresh` | Restart command |
| `scripts/unleash-exit` | Exit without restart |

## Repository Overview

**Unleash** is a wrapper around Anthropic's official Claude Code CLI that adds auto-mode, version management, and a plugin system — without modifying Claude Code itself.

### Key Principles

1. **Claude Code is external** - Installed separately via native binary (GCS) or npm; never bundled or modified
2. **All extensions are plugins** - Custom functionality goes in `plugins/` directory
3. **Configuration over code** - Use profiles (`~/.config/unleash/profiles/`) and `--plugin-dir` for preferences
4. **Auto-mode via hooks** - Stop hook + flag file system, not cli.js patching
5. **Plugin isolation** - Each plugin is self-contained and independently testable

## Repository Structure

```
unleash/
├── src/                         # Rust TUI & CLI source (main entry point)
│   ├── bin/                     # CLI entrypoints
│   └── lib.rs                   # Core logic
├── Cargo.toml                   # Build configuration + version lists
├── scripts/                     # All shell scripts consolidated here
│   ├── install.sh              # Installation script
│   ├── install-remote.sh       # Remote one-line installer
│   ├── unleash-refresh         # Restart command
│   └── unleash-exit            # Exit command
├── plugins/bundled/             # Plugin extensions
│   ├── auto-mode/              # Autonomous operation mode
│   ├── hyprland-focus/         # Window transparency for Hyprland
│   ├── mcp-refresh/            # MCP config change detection
│   ├── process-restart/        # Self-restart hooks and commands

├── docs/                        # Documentation
├── tests/                       # Test scripts
├── .github/workflows/           # CI/CD workflows
└── CLAUDE.md                    # This file - agent instructions
```

## Understanding the Architecture

### Two-Layer Design

1. **Wrapper Layer** (this repository)
   - Rust TUI for profile and version management
   - Launches Claude Code with `--dangerously-skip-permissions`
   - Auto-mode via Stop hook + flag file system
   - Plugin loading via `--plugin-dir`
   - Version management (install, switch, whitelist/blacklist)

2. **Extension Layer** (`plugins/`)
   - Custom functionality
   - Team-specific integrations
   - Workflow automations
   - Each plugin is independent

### Why This Matters for Agents

When working on this repository:

- **Adding features**: Create or modify plugins in `plugins/`
- **TUI/CLI changes**: Modify Rust source in `src/`
- **Configuration changes**: Edit profiles in `~/.config/unleash/profiles/`
- **Version lists**: Edit `Cargo.toml` (whitelist/blacklist sections)
- **Documentation**: Update `README.md` or `docs/extensions/`

**NOTE**: Claude Code is installed separately (via native binary or npm). This repo does not contain or modify Claude Code source.

## Plugin Development Workflow

### When a User Asks for a New Feature

1. **Assess the request**
   - Does this belong in upstream Claude Code? (Suggest they contribute to Anthropic)
   - Is this organization-specific? (Create/update a plugin)
   - Is this configuration? (Update profiles or plugin config)

2. **Create or identify target plugin**
   ```bash
   # New plugin
   mkdir -p plugins/new-feature-name

   # Or extend existing
   cd plugins/existing-plugin
   ```

3. **Implement plugin structure**
   ```
   plugins/my-plugin/
   ├── plugin.json          # Manifest
   ├── index.js             # Main entry point
   ├── README.md            # Documentation
   ├── hooks/               # Lifecycle hooks
   │   ├── pre-command.js
   │   └── post-command.js
   └── tests/               # Plugin tests
       └── index.test.js
   ```

4. **Test the plugin**
   - Create tests in `tests/` directory
   - Test with Claude Code CLI
   - Verify no conflicts with other plugins

5. **Update configuration**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [heiervang-technologies/unleash](https://github.com/heiervang-technologies/unleash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
