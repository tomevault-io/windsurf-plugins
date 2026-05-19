---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`cac` (Coding Agent Config) is a production-grade CLI tool for managing versioned ZIP-based configuration bundles for AI coding assistants. It supports centralized storage via Gokapi backend or local filesystem, enabling configuration portability across hosts and users.

Supports: Claude Code, Codex CLI, Gemini CLI, and Mistral Vibe.

## Quick Start

```bash
# Install
./install.sh

# Configure backend
cp .env.example ~/.config/cac/.env
chmod 600 ~/.config/cac/.env
# Edit ~/.config/cac/.env with backend settings

# Use
cac push                    # Bundle and upload current user's config
cac pull                    # Download and apply globally newest bundle
cac pull [BUNDLE_ID]        # Download and apply specific bundle
cac list                    # List available bundles
cac test                    # Test AI tool API connectivity
```

## Repository Structure

```
bpm-CodingAgentConfigCopy/
├── bin/
│   └── cac                      # Main CLI entrypoint
├── lib/
│   ├── backend_gokapi.sh        # Gokapi REST API integration (7-day max TTL)
│   ├── backend_local.sh         # Local filesystem backend
│   ├── bundle.sh                # ZIP creation/extraction logic
│   ├── check.sh                 # Credential verification with caching
│   ├── config.sh                # Configuration loading (.env)
│   ├── env.sh                   # AI tool environment management (install/update/status)
│   ├── logging.sh               # Logging utilities (info, warn, error, verbose)
│   ├── security.sh              # Permission checks, zip-slip protection
│   ├── tools.sh                 # Tool-specific file mappings
│   ├── update.sh                # Self-update logic (scope detection, version check)
│   └── utils.sh                 # Shared utilities (JSON parsing, retry, filters)
├── tests/
│   ├── run_tests.sh             # Test runner
│   ├── test_bundle.sh           # Bundle tests
│   ├── test_security.sh         # Security validation tests
│   ├── test_integration.sh      # End-to-end tests
│   ├── test_env_settings.sh     # Claude Code settings.json merge tests
│   └── test_update.sh           # Self-update module tests
├── install.sh                   # Bootstrap installer
├── uninstall.sh                 # Clean removal script
├── cpaiagentconfig.sh           # Legacy single-host copy script
├── .env.example                 # Configuration template
└── README.md                    # User documentation
```

## Architecture

### CLI Commands

| Command | Description |
|---------|-------------|
| `cac push [--user USER] [--skip-check]` | Create ZIP bundle from user configs and upload to backend |
| `cac pull [BUNDLE_ID] [--tool TOOL] [--user USER]` | Download and apply bundle (globally newest, filtered, or specific) |
| `cac get` | Silent alias for `cac pull` (backward compatibility) |
| `cac list [--tool TOOL] [--user USER]` | List available bundles with optional filtering |
| `cac check [TOOL] [--user USER]` | Verify AI tool credentials work (real API calls) |
| `cac test [--user USER]` | Alias for check (backward compatibility) |
| `cac update [--check]` | Self-update cac to the latest version |
| `cac env status [--parseable]` | Show AI tool installation status |
| `cac env install [TOOL] [--global] [--yes] [--tmux]` | Install AI tool environments |
| `cac env update [TOOL]` | Update installed AI tools |

### Library Modules

- **config.sh**: Loads `.env` configuration, validates backend settings, checks file permissions
- **security.sh**: User access checks, file permission validation, zip-slip protection, secure temp directories
- **tools.sh**: Maps AI tools to their configuration files (dual registry: credentials + settings), collects/counts existing files
- **bundle.sh**: ZIP creation with correct naming convention, secure extraction with backups
- **backend_local.sh**: Local filesystem storage operations (upload, download, list, get_newest)
- **backend_gokapi.sh**: Gokapi REST API operations (upload, download, list, get_newest, delete); enforces 7-day max TTL
- **check.sh**: Credential verification via real API calls; 5-minute cache, 10-second timeout per provider
- **env.sh**: AI tool environment management; install/update/status for Claude, Codex, Gemini, Mistral Vibe, continuous-claude; `--tmux` flag sets `teammateMode` in settings.json
- **logging.sh**: Structured logging (info, warn, error, verbose, spinner)
- **update.sh**: Self-update logic; detects install scope (user/global), compares local vs remote version, downloads and re-runs install.sh
- **utils.sh**: Shared utilities (JSON field extraction, retry with backoff, filter parsing, command context)

### Bundle Naming Convention

```
CodingAgentConfig_<HOST>_<USER>_<YYMMDD-HHMMSS>.zip
```

### Credential Files (portable across hosts)

| Tool | Files |
|------|-------|
| Claude Code | `.claude.json`, `.claude/.credentials.json` |
| Codex CLI | `.codex/auth.json` |
| Gemini CLI | `.gemini/oauth_creds.json`, `.gemini/google_accounts.json`, `.gemini/settings.json`, `.gemini/state.json`, `.gemini/installation_id`, `.config/gcloud/application_default_credentials.json` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BPMspaceUG/bpm-CodingAgentConfigCopy](https://github.com/BPMspaceUG/bpm-CodingAgentConfigCopy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
