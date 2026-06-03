---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Fujin is a deployment tool for getting projects running on a VPS. It manages app processes using systemd and runs apps behind Caddy reverse proxy. The tool supports both Python packages and self-contained binaries, providing automatic SSL certificates, secrets management, and rollback capabilities.

**Core Philosophy**: Automate deployment while leaving users in full control of their Linux box. It's not a CLI PaaS - users should be able to SSH into their server and troubleshoot.

## Development Commands

### Environment Setup
```bash
# Install dependencies
uv sync

# Run fujin in development mode
uv run fujin --help
```

### Testing
```bash
# Run unit tests (excludes integration tests)
just test
# Or: uv run pytest --ignore=tests/integration -sv

# Run integration tests (requires Docker)
just test-integration
# Or: uv run pytest tests/integration

# Run specific test
just test tests/test_config.py::test_name

# Update inline snapshots (uses inline-snapshot library)
just test-fix

# Review inline snapshot changes
just test-review
```

### Code Quality
```bash
# Format code (ruff + pyproject-fmt)
just fmt

# Type checking
just lint
# Or: uvx mypy .
```

### Documentation
```bash
# Serve docs with live reload
just docs-serve
# Or: uv run --group docs sphinx-autobuild docs docs/_build/html --port 8002 --watch src/fujin
```

### Example Projects
```bash
# Run uv commands in Django example
just djuv [ARGS]

# Generate Django requirements
just dj-requirements

# Run fujin in Django example context
just fujin [ARGS]

# Test with Vagrant VM
just recreate-vm
just ssh
```

### Release Management
```bash
# Bump version and generate changelog
just bumpver [major|minor|patch]

# Generate changelog only
just logchanges

# Build binary distribution (uses PyApp)
just build-bin
```

## Architecture

### Configuration System (`config.py`)

The `Config` class (msgspec struct) is the central configuration object, loaded from `fujin.toml` in the project root.

**Key components:**
- `Config`: Main configuration with app metadata, host config, and deployment settings
- `HostConfig`: SSH connection details, environment files, and deployment target info
- `SecretConfig`: Secret adapter configuration (adapter name, password env var)
- `InstallationMode`: Enum for `python-package` vs `binary` deployment

**Important Config fields:**
- `app_name`: Application name (used for systemd unit naming)
- `app_user`: System user to run the app as (defaults to app_name)
- `version`: App version (defaults to reading from `pyproject.toml`)
- `replicas`: Dict mapping service name to replica count (e.g., `{"worker": 3}`)
- `hosts`: List of `HostConfig` for deployment targets
- `local_config_dir`: Path to `.fujin/` directory (default)

**Important Config properties:**
- `app_dir`: Returns `/opt/fujin/{app_name}` (deployment location)
- `deployed_units`: List of `DeployedUnit` discovered from `.fujin/systemd/`
- `systemd_units`: All unit names that should be enabled/started
- `caddyfile_exists`: Whether `.fujin/Caddyfile` exists
- `caddy_config_path`: Remote Caddy config path (`/etc/caddy/conf.d/{app_name}.caddy`)

**Important behaviors:**
- Version defaults to reading from `pyproject.toml`
- Python version can be read from `.python-version` file if not specified
- Apps are always deployed to `/opt/fujin/{app_name}`
- Systemd unit names follow pattern: `{app_name}-{service}.service` for single replica, `{app_name}-{service}@.service` for multiple replicas

### Service Discovery (`discovery.py`)

The `DeployedUnit` class represents a discovered systemd service from the `.fujin/systemd/` directory.

**DeployedUnit fields:**
- `service_name`: Base name (e.g., "web", "worker")
- `is_template`: True if using `@.service` format for replicas
- `service_file`, `socket_file`, `timer_file`: Paths to unit files
- `template_service_name`: Full systemd name (e.g., `myapp-web@.service`)
- `replica_count`: Number of instances to run
- `instance_service_names`: List of instance names for operations

**Discovery process:**
- Scans `.fujin/systemd/` for `.service` files
- Detects template services (ending with `@`)
- Finds matching `.socket` and `.timer` files
- Generates instance names based on `config.replicas`

### Connection & SSH (`connection.py`)

`SSH2Connection` wraps ssh2-python for executing remote commands. Uses context manager pattern via `connection()` method.

**Key features:**
- Non-blocking I/O with select() for real-time output streaming
- PTY support for interactive sessions (password prompts, shells)
- Automatic sudo password handling via watchers
- UTF-8 incremental decoding to handle split characters across packets
- Directory context manager (`cd()`) for maintaining working directory state
- File upload via SCP (`put()` method)

**PATH handling**: Automatically prepends `~/.cargo/bin` and `~/.local/bin` to PATH to find tools like `uv`.

### Commands Structure (`commands/`)

All commands inherit from `BaseCommand` which provides `config`, `output`, and `connection()` properties. Uses Cappa for CLI parsing.

**Main commands:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Tobi-De/fujin](https://github.com/Tobi-De/fujin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
