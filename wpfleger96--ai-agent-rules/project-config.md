---
trigger: always_on
description: Instructions for AI coding agents working on this repository.
---

# AGENTS.md

Instructions for AI coding agents working on this repository.

## Project Identity

| Aspect | Value |
|--------|-------|
| **PyPI package** | `ai-agent-rules` |
| **CLI command** | `ai-agent-rules` (canonical) or `ai-rules` (alias) |
| **Python module** | `ai_rules` |

The name `ai-rules` was taken on PyPI, so the package is published as `ai-agent-rules`. Both CLI entry points work. Use `ai-agent-rules` as the canonical name; `ai-rules` is kept as a convenience alias.

**Supported agents:** Claude Code, Goose, Gemini CLI, Codex CLI, Amp, Shared (AGENTS.md, skills)

## Quick Commands

```bash
just                          # Lint, format, and type check
just test                     # Run tests
just test-unit                # Run unit tests only
just test-integration         # Run integration tests only
just test-e2e                         # Run E2E tests (subprocess, real CLI; CI: ubuntu via ci.yml, macos/windows via e2e.yml)
uv run ai-agent-rules <cmd>         # Run CLI

# GitHub installation
uv run ai-agent-rules setup --github  # Install from GitHub instead of PyPI

# Key CLI commands
uv run ai-agent-rules install        # Install symlinks
uv run ai-agent-rules status         # Check symlink status (shows diffs)
uv run ai-agent-rules upgrade        # Upgrade to latest (shows changelogs)
uv run ai-agent-rules validate       # Validate config files
uv run ai-agent-rules diff           # Show diffs between repo and installed

# Filtering flags (apply to install, status, diff, uninstall)
#   --agents <list>  Comma-separated agent IDs to target (default: all)
#   --only <list>    Comma-separated component types to target (default: all)
#                    Valid values: config, skills, settings, mcps, plugins,
#                    extensions, completions, tools, source-files
#                    Composes with --agents as an intersection

# Subcommands
uv run ai-agent-rules config show    # Show current config
uv run ai-agent-rules config edit    # Edit user config in $EDITOR
uv run ai-agent-rules override list  # List settings overrides
uv run ai-agent-rules completions install  # Install shell completions
uv run ai-agent-rules profile list   # List available profiles
uv run ai-agent-rules profile switch <name>  # Switch to different profile
```

## Tech Stack

- Python 3.14+ with strict type checking (mypy)
- **uv** for dependency management
- **Click** for CLI framework
- **Rich** for console output
- **pytest** with xdist for parallel testing
- **just** for task automation
- **ruff** for linting and formatting

## Project Structure

```
src/ai_rules/
├── cli/                # Click CLI package (commands, groups, components, helpers)
├── config.py           # Config loading, path parsing, merging, preserved fields
├── profiles.py         # Profile loading and inheritance resolution
├── state.py            # State management (active profile tracking)
├── utils.py            # Deep merge and utility functions
├── symlinks.py         # Symlink operations with backups
├── plugins.py          # Claude Code plugin management via marketplace
├── mcp.py              # MCP server management
├── skills.py           # Shared skills management for Claude Code & Goose
├── claude_extensions.py # Claude extensions (agents, commands, hooks) status
├── completions.py      # Shell completion management
├── agents/
│   ├── base.py         # Abstract Agent base class
│   ├── amp.py          # Amp agent (ampcode.com)
│   ├── claude.py       # ClaudeAgent (settings, MCPs, extensions)
│   ├── codex.py        # CodexAgent (config.toml, MCPs)
│   ├── gemini.py       # GeminiAgent (settings, MCPs)
│   ├── goose.py        # GooseAgent (config, hints, MCPs)
│   └── shared.py       # SharedAgent (AGENTS.md, shared skills)
├── bootstrap/          # GitHub install utilities
│   ├── registry.py     # Tool lifecycle registry (DEPRECATED_TOOLS, ACTIVE_TOOLS) — single source of truth
│   ├── installer.py    # Generic install/uninstall (ensure_tool_installed, ensure_tool_uninstalled)
│   ├── updater.py      # Update checking
│   └── version.py      # Version parsing
└── config/             # Source configs (bundled in package)
    ├── AGENTS.md       # Shared behavioral rules
    ├── chat_agent_hints.md  # Chat agent hints
    ├── mcps.json       # Shared MCP server definitions
    ├── amp/            # Amp configs (AGENTS.md, settings.json)
    ├── claude/         # Claude Code configs (CLAUDE.md, settings.json, mcps.json)
    ├── codex/          # Codex configs (config.toml)
    ├── gemini/         # Gemini configs (GEMINI.md, settings.json)
    ├── goose/          # Goose configs (.goosehints, config.yaml)
    ├── skills/         # **SHARED** skills (symlinked to Claude, Goose, Codex, Amp)
    │   ├── agents-md/, code-reviewer/, continue-crash/, crossfire/
    │   ├── dev-docs/, doc-writer/, pr-creator/, prompt-critique/
    │   ├── prompt-engineer/, test-writer/
    ├── profiles/       # Built-in profiles (default.yaml, personal.yaml, work.yaml); fragments/ for agents_md_file content
    └── buzz/           # Multi-agent Buzz coordinator prompts
tests/
├── fixtures/           # Test fixture files
├── unit/               # No filesystem side effects
├── integration/        # Modifies files/symlinks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wpfleger96/ai-agent-rules](https://github.com/wpfleger96/ai-agent-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
