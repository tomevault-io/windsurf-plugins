---
trigger: always_on
description: This document provides guidance for AI agents working with the labspace-infra codebase.
---

# CLAUDE.md

This document provides guidance for AI agents working with the labspace-infra codebase.

## Project Overview

Labspace Infra is a Docker-based infrastructure for creating interactive educational environments. A Labspace provides:
- A split-screen browser interface (markdown content on left, VS Code IDE on right)
- A fully isolated, Docker-enabled development environment
- Interactive code blocks that can be copied, run, or saved directly from the documentation

Users launch a Labspace with a single command and access it at `http://localhost:3030`.

## Architecture

The system consists of multiple coordinated Docker containers:

```
┌─────────────────────────────────────────────────────────────────┐
│  Interface (interface)                                          │
│  - Express API backend + React/Vite frontend                    │
│  - Renders markdown content, handles code execution             │
│  - Port 3030 (production) or 5173 (dev)                        │
└─────────────────────────────────────────────────────────────────┘
           │
           │ JWT-signed requests via Unix socket
           ▼
┌─────────────────────────────────────────────────────────────────┐
│  Workspace (workspace)                                          │
│  - code-server (VS Code in browser)                            │
│  - Custom labspace-support VS Code extension                    │
│  - Port 8085                                                    │
└─────────────────────────────────────────────────────────────────┘
           │
           │ Docker commands via proxied socket
           ▼
┌─────────────────────────────────────────────────────────────────┐
│  Socket Proxy (socket-proxy)                                    │
│  - Wraps Docker daemon with security controls                   │
│  - Labels all containers for cleanup                            │
│  - Adds containers to labspace network                          │
│  - Remaps mount paths for isolation                             │
└─────────────────────────────────────────────────────────────────┘

Supporting services:
- Configurator: Clones content repos, generates keypairs, runs init scripts
- Host Port Republisher: Forwards ports from user-created containers
- Workspace Cleaner: Removes abandoned labspace resources
```

### Key Volumes

| Volume | Purpose |
|--------|---------|
| `labspace-content` | Project files (user workspace files in `/home/coder/project`) |
| `labspace-instructions` | Labspace instructions (labspace.yaml + markdown files) |
| `labspace-socket-proxy` | Proxied Docker socket |
| `labspace-support` | Security keypairs, metadata, extension socket |

### Volume Mounting and File Paths

The content volumes are mounted differently across services:

| Service | Volume | Mount Point | Access | Purpose |
|---------|--------|-------------|--------|---------|
| **Configurator** | `labspace-content` | `/project` | Read-write | Copies project files from staging |
| | `labspace-instructions` | `/instructions` | Read-write | Copies instruction files from staging |
| **Interface** | `labspace-instructions` | `/labspace/instructions` | Read-only | Reads labspace.yaml and markdown files |
| **Workspace** | `labspace-content` | `/home/coder/project` | Read-write | User's working directory in VS Code |

This separation ensures instructions cannot be modified by users and keeps the workspace clean.

## Content Loading (Configurator)

The configurator service runs once at startup and is responsible for loading content and setting up the environment. It runs as a one-shot container that must complete successfully before other services start.

### Content Sources (in priority order)

The configurator supports multiple ways to load content:

1. **DEV_MODE=true**: Content is initially copied from `/dev-content`, then Compose Watch syncs subsequent changes. Used by `compose.yaml` for infrastructure development.

2. **LOCAL_MODE=true + LOCAL_CONTENT_PATH**: Copies content from a local path. Used when content is bind-mounted.

3. **PROJECT_TAR_PATH**: Extracts content from a base64-encoded tarball. Used for OCI artifact deployments.

4. **PROJECT_CLONE_URL**: Clones content from a git repository. Used by `compose.run.yaml` for production deployments.

**Content Repository Structure:**

Content repositories must follow this directory structure:

```
content-repo/
├── labspace/              # Instructions and configuration
│   ├── labspace.yaml      # Labspace configuration
│   └── *.md               # Markdown instruction files
└── project/               # User workspace files
    └── (code, scripts, etc.)
```

The configurator copies these to separate destinations:
- `/staging/labspace/*` → `/instructions` (mounted as `labspace-instructions` volume)
- `/staging/project/*` → `/project` (mounted as `labspace-content` volume)

If `PROJECT_SUBPATH` is set, only that subdirectory is copied (legacy behavior for repos not following the new structure).

### Init Scripts

After content is staged, the configurator looks for executable scripts in `/init-scripts/` and runs them in order.

This is useful for:
- Downloading additional dependencies
- Pre-building images
- Setting up initial project state

### Setup Sequence

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dockersamples/labspace-infra](https://github.com/dockersamples/labspace-infra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
