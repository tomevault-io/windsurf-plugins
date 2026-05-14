---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Development Commands

### Building
```bash
# Development mode
crystal run src/main.cr

# Production build (static binary)
shards build --release --progress

# Multi-architecture static builds (AMD64 + ARM64)
./build_static.sh

# Docker build (multi-arch)
docker buildx build --platform linux/amd64,linux/arm64 -t tocry .
```

### Testing
```bash
# Run all tests (unit + integration)
make test

# Run unit tests only
crystal spec
# or use convenience script
./run_unit.sh

# Run integration tests only (requires Playwright)
./run_integration.sh

# Run E2E tests (requires Playwright)
cd src/js && npm run test:e2e

# Both tests run automatically on commit (via pre-commit hooks)
```

### Linting and Code Quality
```bash
# Lint code
ameba

# Auto-fix linting issues
ameba --fix
```

### Release Management
```bash
# Full release process (version bump, changelog, Docker upload)
./do_release.sh

# Upload Docker images only
./upload_docker.sh
```

### Installation Script
```bash
# Install ToCry automatically (one-liner, system-wide)
curl -sSL https://tocry.ralsina.me/install.sh | sudo bash

# Install for current user only
curl -sSL https://tocry.ralsina.me/install.sh | bash

# Install with custom options
INSTALL_DIR=$HOME/.local/bin DATA_DIR=$HOME/.local/share/tocry ./install.sh

# Uninstall ToCry
curl -sSL https://tocry.ralsina.me/install.sh | bash -s -- --uninstall

# Show help
./install.sh --help
```

The install script provides:
- Automatic architecture detection (AMD64/ARM64)
- System-wide or user installation
- Systemd service creation (for root installations)
- Data directory setup
- Clean uninstallation
- Comprehensive error handling

## Architecture Overview

ToCry is a Kanban-style TODO application built in Crystal using the Kemal web framework. It's designed as a single-binary, self-hosted application with file-based persistence.

### Core Architecture

1. **Multi-board System**: Users can have multiple Kanban boards, each containing lanes and notes
2. **File-based Storage**: Uses JSON files for persistence (no database required)
3. **User Isolation**: Each user gets their own data directory for multi-tenant support
4. **Static Asset Bundling**: All CSS, JS, and assets are compiled into the binary using BakedFileSystem

### Key Components

- **src/tocry.cr**: Main application module and configuration
- **src/main.cr**: Application entry point and server setup
- **src/board_manager.cr**: Handles multi-board management and persistence with UUID-based naming
- **src/endpoints/**: RESTful API endpoints (boards, lanes, notes, uploads, auth)
- **src/assets/**: Frontend JavaScript and CSS
- **templates/**: ECR templates for server-side rendering

### Domain Models

- **Board**: Kanban board containing lanes with sepia_id-based naming
- **Lane**: Column within a board containing ordered notes
- **Note**: Individual task items with support for Markdown, priority labels (High/Medium/Low), file attachments, start/end dates, and collapsible content

### Configuration System

ToCry uses a unified configuration system that supports three sources with clear precedence:

1. **Command Line Arguments** (highest precedence)
2. **Environment Variables** (with `TOCRY_` prefix)
3. **Configuration File** (YAML or JSON, lowest precedence)

#### Configuration Examples

**Using a configuration file:**
```bash
# Create config.yml
cat > config.yml << EOF
port: 8080
data_path: "/custom/data/path"
safe_mode: true
ai_model: "glm-4-plus"
cache_size: 2000
EOF

# Run with config file
tocry --config config.yml
```

**Using environment variables:**
```bash
export TOCRY_PORT=8080
export TOCRY_DATA_PATH="/custom/data/path"
export TOCRY_SAFE_MODE=true
export TOCRY_AI_MODEL="glm-4-plus"
tocry
```

**Using command line arguments:**
```bash
tocry --port=8080 --data-path="/custom/data/path" --safe-mode --ai-model="glm-4-plus"
```

**Mixed configuration (precedence: CLI > env > config file):**
```bash
# config.yml has port: 8080
export TOCRY_PORT=9000
tocry --config config.yml --port=9999  # Final port will be 9999 (CLI wins)
```

**Configuration file format (YAML):**
See `config-example.yml` for a complete example configuration file.

### Authentication Modes

The application supports three authentication modes (determined automatically based on available credentials):

1. **Google OAuth** (priority 1): Configure via CLI, environment variables, or config file:
   ```bash
   # CLI
   tocry --google-client-id="your-client-id" --google-client-secret="your-secret"

   # Environment variables
   export GOOGLE_CLIENT_ID="your-client-id"
   export GOOGLE_CLIENT_SECRET="your-secret"
   tocry

   # Config file
   google_client_id: "your-client-id"
   google_client_secret: "your-secret"
   ```

2. **Basic Auth** (priority 2): Configure via CLI, environment variables, or config file:
   ```bash
   # CLI
   tocry --auth-user="admin" --auth-pass="password"

   # Environment variables
   export TOCRY_AUTH_USER="admin"
   export TOCRY_AUTH_PASS="password"
   tocry

   # Config file
   auth_user: "admin"
   auth_pass: "password"
   ```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ralsina/tocry](https://github.com/ralsina/tocry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
