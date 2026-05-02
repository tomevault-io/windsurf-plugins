---
trigger: always_on
description: Comprehensive Claude Code skills, agents, and commands for homelab service management.
---

# Claude Homelab - Development Guidelines

Comprehensive Claude Code skills, agents, and commands for homelab service management.

## Glossary

- **Skill**: A Claude Code plugin providing commands and scripts for a specific service (e.g., Plex, Radarr)
- **Agent**: A specialized AI agent for complex workflows (e.g., `notebooklm-specialist`)
- **Command**: A slash command invocable in Claude Code (e.g., `/firecrawl:scrape`, `/homelab:docker-health`)
- **Script**: Executable code in skill `scripts/` directories that performs API calls or system operations
- **Reference**: Detailed documentation in skill `references/` directories (API endpoints, troubleshooting, etc.)
- **Symlink**: Symbolic link connecting this repo to `~/.claude/` for Claude Code discovery
- **SKILL.md**: Claude-facing skill definition with commands, workflows, and examples
- **README.md**: User-facing documentation for setting up and using a skill
- **.env**: Environment file containing credentials (gitignored, NEVER commit)
- **.env.example**: Template credential file (tracked in git, NO secrets)

## Repository Overview

This repository provides production-ready integrations for self-hosted homelab services via a dual-path install:
- **Plugin path** (`/plugin marketplace add jmagar/claude-homelab`) — Claude Code native plugin system
- **Bash path** (`curl -sSL .../install.sh | bash`) — symlinks into `~/.claude/`

- **homelab-core plugin** — agents, commands, prompts, setup wizard, health dashboard (repo root IS the plugin)
- **Service skills** (`skills/`) — 16 service integrations + 2 homelab-core skills (18 total); each skill directory is independently usable
- **Shared library** (`scripts/load-env.sh`) — credential loading, installed to `~/.claude-homelab/`

## Repository Structure

```
claude-homelab/
├── README.md                        # User-facing documentation (comprehensive)
├── CLAUDE.md                        # This file - development guidelines
├── .env.example                     # Credential template (tracked, no secrets)
├── .claude-plugin/
│   ├── marketplace.json             # Plugin catalog (27 plugins)
│   └── plugin.json                  # homelab-core manifest (root IS the plugin)
│
├── agents/                          # homelab-core agents
│   └── notebooklm-specialist.md
│
├── commands/                        # homelab-core slash commands (.md definitions)
│   ├── check.md                     # /check
│   ├── deploy.md                    # /deploy
│   ├── quick-push.md                # /quick-push
│   ├── save-to-md.md                # /save-to-md
│   ├── validate-plan.md             # /validate-plan
│   ├── homelab/                     # /homelab:system-resources, docker-health, disk-space, zfs-health
│   └── notebooklm/                  # /notebooklm:create, ask, source, generate, download, list, research
│
├── prompts/                         # Command prompt definitions (.toml sidecars)
│   ├── check.toml                   # Prompt body for /check
│   ├── deploy.toml                  # Prompt body for /deploy
│   ├── quick-push.toml              # Prompt body for /quick-push
│   ├── save-to-md.toml              # Prompt body for /save-to-md
│   ├── validate-plan.toml           # Prompt body for /validate-plan
│   └── homelab/                     # Prompt bodies for /homelab:* commands
│       ├── disk-space.toml
│       ├── docker-health.toml
│       ├── system-resources.toml
│       └── zfs-health.toml
│
├── docs/references/                 # Shared reference documentation
│   └── security-patterns.md         # Reusable security patterns for scripts
│
├── skills/                          # All service skills (16 services + 2 homelab-core)
│   ├── CLAUDE.md                    # Skill development guidelines
│   ├── homelab-setup/SKILL.md       # /homelab-core:setup — interactive credential wizard
│   ├── homelab-health/
│   │   ├── SKILL.md                 # /homelab-core:health — service health dashboard
│   │   └── scripts/check-health.sh  # Curl-checks all services, outputs JSON
│   └── [service]/                   # e.g., plex/, radarr/, sonarr/, ...
│       ├── SKILL.md                 # Skill definition
│       ├── scripts/                 # Bash/Python/Node API scripts
│       └── references/              # API docs, quick-reference, troubleshooting
│
└── scripts/                         # Install and maintenance scripts
    ├── install.sh                   # Bash path entry point
    ├── setup-creds.sh               # Creates ~/.claude-homelab/.env (both paths)
    ├── setup-symlinks.sh            # Bash path: symlinks skills/ → ~/.claude/skills/
    └── verify.sh                    # Dual-path verification (exits 0 if clean)
```

## Marketplace Repos

The `claude-homelab` marketplace (`.claude-plugin/marketplace.json`) encompasses 11 repositories — 1 core mono-repo plus 10 external plugin repos:

| # | Plugin | Repo | Local Path | Category |
|---|--------|------|------------|----------|
| 1 | **homelab-core** | [jmagar/claude-homelab](https://github.com/jmagar/claude-homelab) | `~/claude-homelab` (this repo) | core |
| 2 | **overseerr-mcp** | [jmagar/overseerr-mcp](https://github.com/jmagar/overseerr-mcp) | `~/workspace/overseerr-mcp` | media |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jmagar/claude-homelab](https://github.com/jmagar/claude-homelab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
