---
trigger: always_on
description: This file guides Claude Code when working with this Ansible-based deployment system for setting up development environments on remote Debian VMs.
---

# CLAUDE.md - Claude Code VM Deployment System

This file guides Claude Code when working with this Ansible-based deployment system for setting up development environments on remote Debian VMs.

## 🎯 Project Purpose

This project automates the deployment of Claude Code and associated development tools to remote Debian VMs. It creates fully-configured development environments with Git, Docker, Kubernetes, and AI-enhanced capabilities through MCP servers.

**Key Point**: This runs on your LOCAL machine to deploy TO remote VMs. It does NOT run on the target VMs themselves.

## 🏗️ Architecture Overview

### Project Structure
```
claude-code-vm/
├── config/                     # Default configuration templates
│   ├── env.example            # Template for credentials and API keys
│   ├── mcp-servers.template.json  # MCP server definitions
│   ├── git-repos.env.example  # Git repository management template
│   ├── CLAUDE.common.md       # Base CLAUDE.md template (shared by all)
│   ├── CLAUDE.minimal.md      # Minimal deployment template
│   ├── CLAUDE.enhanced.md     # Enhanced deployment with MCP/Docker
│   ├── CLAUDE.containerized.md # Containerized with Docker Compose
│   └── CLAUDE.full.md         # Full deployment with Kubernetes
├── ansible/
│   ├── playbooks/             # Main orchestration
│   │   ├── site.yml          # Primary deployment playbook
│   │   └── validate.yml      # Validation playbook
│   └── roles/                 # Modular components
│       ├── common/           # System preparation
│       ├── git/              # Git + credential management
│       ├── docker/           # Docker CE installation
│       ├── nodejs/           # Node.js 22 LTS
│       ├── claude-code/      # Claude Code CLI
│       ├── kubernetes/       # k8s tools (kubectl, k3s/kind)
│       ├── mcp/              # MCP server configuration
│       └── claude-config/    # CLAUDE.md deployment and inheritance
└── Makefile                   # User interface for all operations
```

### Configuration Defaults
- **ENV_FILE**: `config/.env` (Git credentials, API keys)
- **MCP_FILE**: `config/mcp-servers.json` (MCP server definitions)
- **GIT_CONFIG_FILE**: Same as ENV_FILE (repository definitions)

## 🚀 Essential Commands

### Initial Setup
```bash
make setup                    # Create config files from templates
make check-config            # Validate configuration
make test-connection VM_HOST=<ip> TARGET_USER=<user>  # Test SSH
```

### Deployment Tiers
```bash
# Tier 1: Minimal (Git + Node.js + Claude Code)
make deploy-baseline VM_HOST=<ip> TARGET_USER=<user>

# Tier 2: Enhanced (+ MCP servers + Docker)
make deploy-enhanced VM_HOST=<ip> TARGET_USER=<user>

# Tier 3: Containerized (+ Docker Compose + shell enhancements)
make deploy-containerized VM_HOST=<ip> TARGET_USER=<user>

# Tier 4: Full (+ Kubernetes with k3s/KIND)
make deploy-full VM_HOST=<ip> TARGET_USER=<user>
```

### Common Operations
```bash
# Validate deployment
make validate VM_HOST=<ip> TARGET_USER=<user>

# Deploy CLAUDE.md and settings.json configuration
make deploy-claude-config VM_HOST=<ip> TARGET_USER=<user>

# Deploy only MCP servers
make deploy-mcp VM_HOST=<ip> TARGET_USER=<user>

# Deploy Git repositories
make deploy-git-repos VM_HOST=<ip> TARGET_USER=<user>

# Clean temporary files
make clean
```

## 🔧 Working with Configuration

### Using External Config Files
```bash
# Use configs from another location
make deploy-enhanced VM_HOST=<ip> TARGET_USER=<user> \
  ENV_FILE=/path/to/.env \
  MCP_FILE=/path/to/mcp-servers.json
```

### CLAUDE Configuration Management
```bash
# Deploy with auto-detected template
make deploy-claude-config VM_HOST=<ip> TARGET_USER=<user>

# Deploy with specific template
make deploy-claude-config VM_HOST=<ip> TARGET_USER=<user> \
  CLAUDE_CONFIG_TEMPLATE=config/CLAUDE.full.md

# Force override existing CLAUDE.md
make deploy-claude-config VM_HOST=<ip> TARGET_USER=<user> \
  CLAUDE_CONFIG_FORCE_OVERRIDE=true
```

### Git Repository Management
```bash
# Simple format in .env or git config file
GITHUB_URL=https://github.com/user/repo.git
# OR
GIT_REPO_1_URL=https://github.com/user/repo.git
GIT_REPO_1_BRANCH=main

# Deploy with repository cloning
make deploy-enhanced VM_HOST=<ip> TARGET_USER=<user> \
  MANAGE_GIT_REPOSITORIES=true
```

## 📝 AI Agent Guidelines

When modifying this project:

1. **Use Make targets** - Don't run ansible-playbook directly unless necessary
2. **Test changes** - Always run `make check-config` before deployment
3. **Configuration priority**:
   - Command-line parameters override everything
   - config/ directory contains defaults
   - Never hardcode sensitive information
4. **Error handling** - Check the colored output from Make commands
5. **Validation** - Always run `make validate` after deployments

### Common Tasks

**Adding a new MCP server**:
1. Edit the MCP template: `config/mcp-servers.template.json`
2. Add any required API keys to `config/env.example`
3. Update documentation in README.md

**Debugging deployment issues**:
```bash
# Use verbose mode
make deploy-enhanced VM_HOST=<ip> TARGET_USER=<user> VERBOSE=vv

# Check logs
tail -f deployment.log

# Test specific components

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [intelligentcode-ai/claude-code-vm](https://github.com/intelligentcode-ai/claude-code-vm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
