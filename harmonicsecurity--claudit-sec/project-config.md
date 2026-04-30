---
trigger: always_on
description: CLAUDIT-SEC is a read-only, single-file security audit tool for macOS that inspects Claude Desktop and Claude Code configuration, scheduled tasks, extensions, plugins, skills, permissions, and runtime state.
---

# CLAUDIT-SEC — Claude Security Audit Tool

## Project Overview

CLAUDIT-SEC is a read-only, single-file security audit tool for macOS that inspects Claude Desktop and Claude Code configuration, scheduled tasks, extensions, plugins, skills, permissions, and runtime state.

- **`claude_audit.sh`** — Zsh (~1900 lines, requires `jq`) — designed for MDM/CrowdStrike RTR style deployment

This project is built and maintained using [Claude Code](https://docs.anthropic.com/en/docs/claude-code). This file provides architecture context and development guidelines that Claude Code uses when working on the codebase.

## Architecture

The script follows this structure:

1. **Preflight checks** — OS validation, dependency checks
2. **User context detection** — single user, explicit `--user`, or all-users scan
3. **Session directory discovery** — finds `local-agent-mode-sessions/org/user/` paths
4. **14 data collectors** — each reads specific config files and populates findings
5. **Finding aggregation** — counts by severity (WARN, INFO, REVIEW)
6. **3 output renderers** — ASCII (ANSI color + Unicode tables), HTML (dark theme), JSON (SIEM-ready)

### Data Collectors

| Collector | Source Files | Key Findings |
|-----------|-------------|-------------|
| Desktop Settings | `claude_desktop_config.json` → preferences | keepAwakeEnabled, allowAllBrowserActions, menuBar, sidebar |
| Cowork Settings | `cowork_settings.json`, `config.json` | scheduledTasks, webSearch, networkMode |
| MCP Servers | `claude_desktop_config.json` → mcpServers | Server names, commands, env var keys |
| Plugins | `installed_plugins.json`, remote manifest, marketplace cache | installed/remote/cached plugins |
| Plugin Hooks | `hooks/hooks.json` in plugin directories | Shell commands on lifecycle events |
| Connectors | `local_*.json` → remoteMcpServersConfig, `.mcp.json` | web/desktop/not_connected, egressAllowedDomains, disabledMcpTools |
| Skills | User skills (6 paths), installed plugin skills (3 paths) — see Key Paths | SKILL.md frontmatter parsing |
| Scheduled Tasks | `scheduled-tasks.json` | Cron expressions with english translation |
| Extensions (DXT) | `extensions-installations.json` | Signature status, dangerous tools |
| Extension Settings | `Claude Extensions Settings/*.json` | Allowed directories |
| Blocklist | `extensions-blocklist.json` | Governance control presence |
| Claude Code Settings | `~/.claude/settings.json` | Permission grants |
| Runtime | pgrep, pmset, crontab, LaunchAgents | Running processes, sleep assertions |
| Cookies | `Cookies`, `Cookies-journal` | Presence and permissions |

### Key Paths

```
~/Library/Application Support/Claude/          # Claude Desktop dir
  claude_desktop_config.json                   # Desktop prefs + MCP servers
  config.json                                  # App config (network mode, allowlists, device ID)
  extensions-installations.json                # DXT extensions
  extensions-blocklist.json                    # Extension governance
  Claude Extensions Settings/*.json            # Per-extension settings
  local-agent-mode-sessions/<org>/<user>/      # Session directories
    cowork_settings.json                       # Cowork preferences
    scheduled-tasks.json                       # Scheduled tasks
    cowork_plugins/installed_plugins.json       # Installed plugins
    cowork_plugins/marketplaces/               # Marketplace catalog
    cowork_plugins/cache/                      # Downloaded plugin cache
    remote_cowork_plugins/manifest.json        # Org-deployed plugins
    local_*.json                               # Session files with connector state
~/.claude/settings.json                        # Claude Code settings
~/.claude/skills/<skill>/SKILL.md              # Claude Code user skills (manually uploaded)
~/.claude/plugins/marketplaces/<mp>/           # Claude Code marketplace plugin skills
  plugins/<plugin>/skills/<skill>/SKILL.md
  external_plugins/<plugin>/skills/<skill>/SKILL.md
~/Documents/Claude/Scheduled/<skill>/SKILL.md  # Scheduled task skills

Skill scanning paths (collect_skills):
  User skills (source: "user"):
    ~/Documents/Claude/Scheduled/*/SKILL.md                         # Scheduled tasks
    skills-plugin/<uid>/<oid>/skills/*/SKILL.md                     # Cowork-created skills
    ~/.skills/skills/*/SKILL.md                                     # Alt skills dir
    ~/.claude/skills/*/SKILL.md                                     # Claude Code user skills
    <session>/local_*/.claude/skills/*/SKILL.md                     # Session-local skills

  Installed plugin skills (source: "plugin"):
    <session>/remote_cowork_plugins/*/skills/*/SKILL.md             # Org-deployed plugins
    <session>/cowork_plugins/cache/<mp>/<plugin>/<ver>/skills/*/    # Cached (installed) plugins
    ~/.claude/plugins/marketplaces/<mp>/{plugins,external_plugins}/ # Claude Code marketplace

  NOT scanned (marketplace catalog, not installed):
    <session>/cowork_plugins/marketplaces/                          # Full Cowork catalog
```

## Shell Script (`claude_audit.sh`)

### Zsh Compatibility


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HarmonicSecurity/claudit-sec](https://github.com/HarmonicSecurity/claudit-sec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
