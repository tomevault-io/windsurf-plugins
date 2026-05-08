---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **Note:** `install.sh` copies `claude-md-master/CLAUDE.md` to `~/.claude/CLAUDE.md` on every run (always overwrites). The "Global Claude Code Configuration" section below is kept in sync manually — edit `claude-md-master/CLAUDE.md` to change what gets installed globally.

## Repository Overview

`claude-local-starter` is a single-script bootstrap installer that replicates a full Claude Code environment onto any machine. It is not a library or application — the primary deliverable is `install.sh`.

### Key Files

| File | Purpose |
|------|---------|
| `install.sh` | The installer — idempotent, safe to re-run |
| `scripts/token-audit.sh` | Snapshot + restore extension states for token cost measurement |
| `scripts/enable-safe-yolo.sh` | Add auto-approve permissions + `acceptEdits` mode to a repo |
| `scripts/disable-safe-yolo.sh` | Remove safe-yolo permissions from a repo |
| `scripts/config/claude-safe-yolo-permissions.txt` | Single source of truth for safe-yolo allow/deny/defaultMode |
| `claude-md-master/CLAUDE.md` | Source of truth for `~/.claude/CLAUDE.md` — always overwrites on install |
| `settings.json` | Source of truth for plugins, env vars, MCP servers, hooks — merged into `~/.claude/settings.json` |
| `commands/` | Slash command definitions synced to `~/.claude/commands/` |
| `skills/` | Custom skill directories synced to `~/.claude/skills/` |
| `claude-local-starter.html` | Visual dashboard — open from `~/.claude/` after install (data is injected there) |

### Installer Modes

```bash
bash install.sh                  # safe update — merges, never overwrites
bash install.sh --clean-install  # full overwrite from repo (backs up first)
bash install.sh --dry-run        # preview all actions, no changes written
```

### What install.sh Does (in order)

1. Pre-flight checks (git, node, npm, python3, bun)
2. Interactive prompts — preserve CLAUDE.md? preserve MCP/plugin/skill states? (update mode only)
3. Backup mutable components (only on `--clean-install`)
4. Create `~/.claude/` directory structure
5. Install/merge `settings.json`
6. Install `CLAUDE.md` from `claude-md-master/` (always overwrites)
7. Install skills via `npx skills add` (patched with `disable-model-invocation: true`)
8. Clone/update `blader/humanizer` skill
9. Install `uipro-cli` globally
10. Install LSP binaries (typescript-language-server, pyright, gopls, rust-analyzer, jdtls) — pyright-lsp and typescript-lsp enabled by default; others installed but disabled
11. Install `@playwright/cli`, run `playwright-cli install --skills`, install Chromium
12. Register MCP servers via `npx gitnexus setup`
13. Write `~/.claude/plugin_commands.sh` (manual paste into Claude Code)
14. Create `~/.claude-work/` structure and templates
15. Write shell aliases/functions to `~/.zshrc` / `~/.bashrc`
16. Sync `commands/`, `skills/`, HTML dashboard → `~/.claude/` (with settings.json + CLAUDE.md injected into dashboard)

### Shell Functions Installed

After running the installer, these functions are available in your shell:

- `cw` / `claude-work` — launch Claude with `~/.claude-work` context (exit and run `claude` to go without)
- `enable-skill <name>` / `disable-skill <name>` — toggle `disable-model-invocation` in SKILL.md
- `list-skills` — show all skills with context-on/off state
- `enable-plugin <name>` / `disable-plugin <name>` — toggle `enabledPlugins` in settings.json
- `list-plugins` — show all plugins with enabled/disabled state
- `enable-mcp <name>` / `disable-mcp <name>` — toggle `disabled` flag in settings.json mcpServers
- `list-mcps` — show all MCP servers with enabled/disabled state
- `list-commands` — list all slash commands

### MCP Servers Configured

All disabled by default except `context7`. Use `enable-mcp <name>` to activate.

| Server | Type | Purpose |
|--------|------|---------|
| `context7` | URL | Live documentation fetching (enabled by default) |
| `gitnexus` | Command | Codebase knowledge graph (run `npx gitnexus analyze` per repo) |
| `context-mode` | Command | Context-mode plugin support |
| `claude-mem` | URL | Session memory (requires claude-mem plugin) |
| `filesystem` | Command | Local file access (`${HOME}`) |
| `supabase` | URL | Database queries |
| `vercel` | URL | Deploy and logs |

### Adding Custom Skills

Place skill directories under `skills/` — each needs a `SKILL.md`. The installer syncs them to `~/.claude/skills/` without overwriting existing customisations.

### Bundled Skills

| Skill | Purpose |
|-------|---------|
| `aidlc-tracking` | Canonical formats for all project tracking files |
| `review-council` | Multi-persona architecture review council |
| `ppt-creator` | Personal presentation builder — research, narrative, design system, QA pipeline |

### Adding Custom Commands

Place `.md` files under `commands/` — they sync to `~/.claude/commands/` and become available as `/command-name` inside Claude Code.

### Bundled Commands

| Command | Purpose |
|---------|---------|
| `/init-repo` | Bootstrap gitnexus + CLAUDE.md + AIDLC tracking files for a new repo |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [indranildchandra/claude-local-starter](https://github.com/indranildchandra/claude-local-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
