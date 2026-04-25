---
trigger: always_on
description: This repository contains a Home Assistant custom integration that brings GitHub Copilot AI capabilities to Home Assistant. It enables voice assistants and AI-powered tasks by implementing a conversation agent powered by the GitHub Copilot SDK.
---

# GitHub Copilot Instructions for GitHub Copilot Home Assistant Integration

## Project Overview

This repository contains a Home Assistant custom integration that brings GitHub Copilot AI capabilities to Home Assistant. It enables voice assistants and AI-powered tasks by implementing a conversation agent powered by the GitHub Copilot SDK.

## Technology Stack

- **Platform**: Home Assistant Custom Integration
- **Language**: Python 3.11+
- **Key Dependencies**:
  - `homeassistant` - Core Home Assistant framework
  - `github-copilot-sdk` - Copilot SDK client library (uses Copilot CLI runtime)
- **Development Tools**:
  - Ruff for linting (configured in `.ruff.toml`)
  - Dev container support (`.devcontainer.json`)

## Code Style and Standards

### Python Standards
- Use type hints with `from __future__ import annotations`
- All I/O operations must be async/await
- Follow Home Assistant coding conventions
- Use Ruff for linting: `ruff check custom_components/github_copilot/`
- Auto-fix issues with: `ruff check --fix custom_components/github_copilot/`

### Naming Conventions
- Use descriptive variable and function names
- Class names: PascalCase
- Functions and variables: snake_case
- Constants: UPPER_SNAKE_CASE

### Documentation
- Add docstrings to all public functions and classes
- Update README.md for user-facing changes
- Keep README.md and CONTRIBUTING.md aligned for technical documentation
- Keep code comments minimal but meaningful
- **Maintain addon/CHANGELOG.md** when making changes to the bridge add-on (Dockerfile, run.sh, config.yaml, build.yaml)
- Follow [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) format for changelog entries

## Project Structure

```
custom_components/github_copilot/
├── __init__.py           # Integration entry point
├── api.py                # GitHub Copilot SDK client
├── config_flow.py        # Configuration UI flow
├── conversation.py       # Conversation agent implementation
├── coordinator.py        # Data update coordinator
├── const.py              # Constants and configuration
└── data.py               # Data models

addon/                    # GitHub Copilot Bridge Add-on
├── Dockerfile            # Container image definition with Copilot CLI
├── config.yaml           # Add-on metadata and configuration schema
├── run.sh                # Server startup script with auth and retry logic
├── build.yaml            # Multi-arch build configuration
└── CHANGELOG.md          # Add-on version history and changes
```

## Key Components

### SDK Client (`api.py`)
- All methods should be async
- Use the GitHub Copilot SDK client
- Implement proper error handling with custom exceptions
- Include connection testing in `async_test_connection()`

### Conversation Agent (`conversation.py`)
- Extends Home Assistant's `ConversationEntity`
- Implements `async_process()` for message handling
- Maintains conversation history within sessions
- Handles errors gracefully with user-friendly messages

### Configuration Flow (`config_flow.py`)
- Validates credentials during setup
- Provides clear error messages for users
- Supports optional configuration parameters (model, cli_url)
- Follows Home Assistant's config flow patterns
- **cli_url parameter**: Enables connection to remote Copilot CLI server (bridge add-on)
- **Mutual exclusion**: When `cli_url` is provided, `github_token` is NOT passed to SDK (remote server handles auth)

### Bridge Add-on (`addon/`)
- **Purpose**: Runs GitHub Copilot CLI as a headless server for Home Assistant OS users
- **Dockerfile**:
  - Uses Debian Bullseye base for native glibc support (required by Copilot CLI)
  - Pins Copilot CLI version (currently v1.0.13) with SHA256 verification
  - Supports amd64 and aarch64 architectures
- **run.sh**:
  - Authenticates CLI via GH_TOKEN environment variable
  - Implements feature detection for optional CLI flags (--bind, --no-auto-update, --log-level)
  - Includes retry mechanism (up to 5 attempts with 5-second delays)
  - Hardened auth probe with timeout protection
- **Current version**: v3.8.3
- **Server port**: 8000 (internal Supervisor network only)
- **When to update**: Bump version in `config.yaml` when making significant changes to Dockerfile or run.sh

## Recent Important Changes (March-April 2026)

### SDK Installation Fix (Current)
- **Integration**: Now uses patched `github-copilot-sdk==0.1.22+ha` wheel from repository
- **Fixes**: Home Assistant OS installation failure due to manylinux_2_28 wheel incompatibility
- **Installation**: Automatic via URL requirement in `manifest.json` pointing to `wheels/github_copilot_sdk-0.1.22+ha-py3-none-any.whl`
- **Benefits**:
  - Universal `py3-none-any` wheel works on all platforms including HA OS
  - Protocol v3 support (patched from 0.1.22 source)
  - No manual installation required
  - No glibc version constraints

### SDK Version and HA OS Compatibility
- **Problem**: SDK 0.1.23+ only ship manylinux_2_28 wheels requiring glibc ≥ 2.28; Home Assistant OS cannot install these

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tserra30/Github-Copilot-SDK-integration](https://github.com/tserra30/Github-Copilot-SDK-integration) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
