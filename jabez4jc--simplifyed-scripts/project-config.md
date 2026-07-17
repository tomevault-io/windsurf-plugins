---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a comprehensive collection of bash scripts for managing OpenAlgo trading platform instances on Ubuntu/Debian servers. The scripts automate multi-instance deployment, configuration, monitoring, backup, and updates.

## Architecture

**Core Components:**

1. **quick-setup.sh** - Single instance setup script
   - Automated complete setup in one command
   - Configures 4GB swap automatically
   - Includes all system packages, dependencies, SSL, Nginx, systemd service
   - Interactive prompts for domain, broker, and credentials
   - Best for single instance deployments and quick testing

2. **multi-install.sh** - Main orchestration script
   - Validates system prerequisites (Python 3, pip, uv)
   - Manages system-wide dependencies (nginx, certbot, firewall)
   - Creates isolated instance directories at `/var/python/openalgo-flask/openalgo<N>/`
   - Generates unique configurations per instance (ports, domains, databases)
   - Creates systemd services and Nginx reverse proxy configs
   - Handles SSL certificate generation via Let's Encrypt

3. **update_swap_4gb.sh** - Fixed swap utility
   - Creates or replaces fixed 4GB swap space to prevent OOM during broker authentication

4. **oa-configure-swap.sh** - Flexible swap utility
   - Interactive or command-line driven swap configuration (1-512 GB)
   - Validates disk space before allocation
   - Displays current swap configuration and filesystem usage
   - Includes confirmation prompts and safe reconfiguration

5. **oa-restart.sh** - Instance management (manual)
   - Discovers running instances via systemd
   - Provides interactive menu for restarting single or all instances
   - Auto-reloads Nginx after restart

5a. **setup-daily-restart.sh** - Automated restart scheduler
   - Sets up cron job for daily automatic restart at 8 AM IST
   - Creates restart script at `/usr/local/bin/openalgo-daily-restart.sh`
   - Creates log file at `/var/log/openalgo-daily-restart.log`
   - Verifies/sets system timezone to Asia/Kolkata
   - Provides easy modification commands for restart time

6. **oa-uninstaller.sh** - Cleanup utility
   - Removes instances with full cleanup (service, directories, SSL certs, nginx config)
   - Includes confirmation prompts to prevent accidental deletion

7. **oa-health-check.sh** - Monitoring utility
   - Multi-category health checks (service, ports, configuration, databases, filesystem, logs, connectivity)
   - System-wide health assessment (Nginx, firewall, swap, load)
   - Exit codes for automation (0=healthy, 1=warning, 2=critical)
   - Supports single instance, all instances, or system-only checks

8. **oa-backup.sh** - Backup & restore utility
   - Quick backups (env + databases + configs) with optional GPG encryption
   - Full backups (complete instance archive)
   - Selective restore with current data preservation
   - Automatic cleanup of old backups (configurable retention)
   - Supports single instance, all instances, or specific backup operations

9. **oa-update.sh** - Smart update utility
   - Version-aware .env merging using `ENV_CONFIG_VERSION` field
   - Selective updates (only merge .env when version changes)
   - Pre-update automatic backup
   - Dependency updates via `uv sync`
   - Dry-run mode to preview updates
   - Rollback capability to pre-update backup

10. **make-executable.sh** - Setup utility
   - Finds all `.sh` files in repository automatically
   - Makes them executable with single command
   - Reports success/failure for each script
   - Provides summary and lists all available scripts
   - Simplifies initial setup process

## Key Implementation Patterns

**Instance Isolation:**
- Each instance uses a unique port range (Flask: 5000+N, WebSocket: 8765+N, ZMQ: 5555+N)
- Separate SQLite databases per instance (openalgo{N}.db, latency{N}.db, logs{N}.db)
- Unique session/CSRF cookie names (session{N}, csrf_token{N}) to prevent cross-instance pollution
- Individual systemd services (openalgo{N}) with separate Unix sockets

**Broker Integration:**
- Validates broker names against hardcoded list (30 supported brokers)
- Special handling for XTS-based brokers that require market data API credentials
- Broker credentials injected via `.env` file during installation

**Configuration Management:**
- Uses `.env` file from cloned OpenAlgo repository as template
- Sed-based replacements to update environment variables
- Order of replacements matters (domain → ports → credentials → keys)

**Error Handling:**
- `check_status()` function aborts on any command failure
- Logging via `log_message()` function with color codes
- All logs saved to `logs/install_multi_TIMESTAMP.log`

## Key Features

**Version-Aware .env Updates (oa-update.sh):**
- Reads `ENV_CONFIG_VERSION = 'X.Y.Z'` from both old and new `.sample.env`
- If versions match: skips merge (code-only updates)
- If versions differ: intelligently merges configuration
- Preserves instance-specific settings: ports, broker, credentials, keys, cookies
- Includes custom variables not in template
- Falls back to MD5 hash comparison if version field missing


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jabez4jc/Simplifyed-Scripts](https://github.com/jabez4jc/Simplifyed-Scripts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
