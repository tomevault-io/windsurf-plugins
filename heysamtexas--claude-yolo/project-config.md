---
trigger: always_on
description: This file provides guidance to Claude Code when working in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working in this repository.

## Project Overview

Docker-based Claude Code environment with safety features (secrets scanning, git hooks, command logging). Designed to protect both the host machine AND inexperienced users (sales engineers, FDEs, CTOs) from well-intentioned but potentially dangerous actions, while giving Claude Code maximum autonomy.

## Available Tools

**Core:** git, gh, jq, ripgrep (rg), vim, nano, tmux, htop, tree
**Cloud:** aws, az, gcloud
**Kubernetes:** kubectl, helm, k9s, docker, docker-compose
**IaC:** terraform, tfsec
**Databases:** psql, mysql, redis-cli, mongosh
**Python:** uv (primary package manager), ruff, mypy, pytest, bandit, pre-commit
**Security:** gitleaks, detect-secrets, trivy
**Networking:** tailscale, openvpn, cloudflared, ttyd
**Utilities:** yq, httpie (http), curl, wget
**Build:** make, cmake, gcc/g++, Node.js 20

## File Structure

```
claude-yolo/
├── src/claude_yolo/           # Python package (CLI tool)
├── terraform/                 # Infrastructure as Code
│   └── azure/                # Azure deployment modules
│       ├── acr/              # Azure Container Registry
│       ├── aci/              # Azure Container Instances
│       ├── vm/               # Azure Virtual Machines
│       └── scripts/          # Helper scripts (push-to-acr.sh)
├── examples/                  # Demo applications
│   └── fastapi-hello-world/  # Simple FastAPI demo
├── demos/                     # Sales engineering demos
│   └── sales-engineering/    # SE demo scripts and guides
├── docs/                      # Documentation
├── tests/                     # Test suite
└── .github/workflows/         # CI/CD pipelines
```

## Safety Constraints

**Container Isolation:**
- Runs as non-root user (developer, UID 1001)
- Resource limits: 2 CPU cores, 4GB RAM (configurable via `.env`)
- Network isolation via Docker bridge
- No Docker-in-Docker access

**Git Safety Hooks:**
- Pre-commit: Scans for secrets (gitleaks, detect-secrets), large files, sensitive patterns
- Pre-push: Prevents force push to main/master, warns on direct pushes
- Auto-configured via `core.hooksPath`

**Pre-commit Framework:**
- Secrets detection (gitleaks, detect-secrets)
- Python linting and formatting (ruff)
- Security scanning (bandit)
- File safety checks (large files, merge conflicts)

**Logging:**
- `/logs/commands/` - All shell commands with timestamps
- `/logs/claude/` - Claude Code session logs
- `/logs/git/` - Git operations
- `/logs/safety/` - Safety check results
- All logs shared with host via volume mount

**Claude Code Mode:**
- BypassPermissions mode enabled by default (`config/.claude/settings.local.json`)
- True YOLO mode - autonomous operation without permission prompts
- Safety provided by container isolation and hooks

## Networking Modes

The container supports two networking modes in `docker-compose.yml`. **Claude should recommend the appropriate mode based on user needs:**

### Mode 1: Bridge Networking (Default)
- **When to recommend:** Multi-container setups (databases, Redis, microservices)
- **Pros:** Docker DNS, network isolation, can join custom networks
- **Cons:** MCP OAuth callbacks won't work (requires manual port forwarding)
- **Config:** Uses `networks: - claude-network` (default in docker-compose.yml)

### Mode 2: Host Networking (For MCP OAuth)
- **When to recommend:** Single-container setup + user needs MCP server authentication (Atlassian, GitHub MCP servers)
- **Pros:** All ports accessible, MCP OAuth works seamlessly, zero overhead
- **Cons:** Cannot join Docker networks, no service discovery, multi-container setups won't work
- **Config:** Use `claude-yolo run --mcp` flag (automatically applies host networking)
  - **Advanced:** Manually edit docker-compose.yml - uncomment `network_mode: "host"` and comment out `networks:`

### MCP OAuth Technical Background
Claude Code uses random ephemeral ports (49152-65535) for OAuth callbacks. Exposing this full range causes:
- Container startup hangs (hours) or complete failure
- 16GB+ RAM consumption
- Docker creates 16,384 iptables rules or docker-proxy processes

**Decision tree for users:**
- Planning to add databases/microservices? → Bridge mode (default)
- Need MCP OAuth + single container only? → Host mode
- Unsure? → Start with bridge mode (default)

See [GitHub Issue #2527](https://github.com/anthropics/claude-code/issues/2527) for Claude Code's OAuth port limitation and [Docker Issue #14288](https://github.com/moby/moby/issues/14288) for port range performance issues.

## File Boundaries

**Safe to edit:** `/workspace/*`, `/config/*`, `/scripts/*`, `Dockerfile`, `docker-compose.yml`, `.env`
**Read-only:** `/opt/config-templates/*`, `/logs/*` (view only)
**Never touch:** System directories, volume mount points

## Volume Mounts

- `/home/developer` - Host home directory bind mount (configurable via `HOST_HOME`, default: `./home`)
- `/workspace` - Host workspace directory (configurable via `HOST_WORKSPACE`)
- `/logs` - Shared logs directory (configurable via `HOST_LOGS`)
- `/mnt/host-gitconfig` - Optional mount for host git config


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/heysamtexas) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
