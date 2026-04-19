---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is **n8n-install**, a Docker Compose-based installer that provides a comprehensive self-hosted environment for n8n workflow automation and numerous AI/automation services. The installer includes an interactive wizard, automated secret generation, and integrated HTTPS via Caddy.

### Core Architecture

- **Profile-based service management**: Services are activated via Docker Compose profiles (e.g., `n8n`, `flowise`, `monitoring`). Profiles are stored in the `.env` file's `COMPOSE_PROFILES` variable.
- **No exposed ports**: Services do NOT publish ports directly. All external HTTPS access is routed through Caddy reverse proxy on ports 80/443.
- **Shared secrets**: Core services (Postgres, Valkey (Redis-compatible, container named `redis` for backward compatibility), Caddy) are always included. Other services are optional and selected during installation.
- **Queue-based n8n**: n8n runs in `queue` mode with Redis, Postgres, and dynamically scaled workers (`N8N_WORKER_COUNT`).

### Key Files

- `Makefile`: Common commands (install, update, logs, etc.)
- `docker-compose.yml`: Service definitions with profiles
- `Caddyfile`: Reverse proxy configuration with automatic HTTPS
- `.env`: Generated secrets and configuration (from `.env.example`)
- `scripts/install.sh`: Main installation orchestrator (runs numbered scripts 01-08 in sequence)
- `scripts/utils.sh`: Shared utility functions (sourced by all scripts via `source "$(dirname "$0")/utils.sh" && init_paths`)
- `scripts/01_system_preparation.sh`: System updates, firewall, security hardening
- `scripts/02_install_docker.sh`: Docker and Docker Compose installation
- `scripts/git.sh`: Git utilities (sync with origin, branch detection, configuration)
- `scripts/03_generate_secrets.sh`: Secret generation and bcrypt hashing
- `scripts/04_wizard.sh`: Interactive service selection using whiptail
- `scripts/05_configure_services.sh`: Service-specific configuration logic
- `scripts/databases.sh`: Creates isolated PostgreSQL databases for services (library)
- `scripts/telemetry.sh`: Anonymous telemetry functions (Scarf integration)
- `scripts/06_run_services.sh`: Starts Docker Compose stack
- `scripts/07_final_report.sh`: Post-install credential summary
- `scripts/08_fix_permissions.sh`: Fixes file ownership for non-root access
- `scripts/generate_n8n_workers.sh`: Generates dynamic worker/runner compose file
- `scripts/update.sh`: Update orchestrator (syncs with origin and updates images)
- `scripts/update_preview.sh`: Preview available updates without applying (dry-run)
- `scripts/doctor.sh`: System diagnostics (DNS, SSL, containers, disk, memory)
- `scripts/apply_update.sh`: Applies updates after git sync
- `scripts/docker_cleanup.sh`: Removes unused Docker resources (used by `make clean`)
- `scripts/download_top_workflows.sh`: Downloads community n8n workflows
- `scripts/import_workflows.sh`: Imports workflows from `n8n/backup/workflows/` into n8n (used by `make import`)
- `scripts/restart.sh`: Restarts services with proper compose file handling (used by `make restart`)
- `scripts/setup_custom_tls.sh`: Configures custom TLS certificates (used by `make setup-tls`); supports `--remove` to revert to Let's Encrypt
- `start_services.py`: Python orchestrator for service startup order, builds Docker images, handles external services (Supabase/Dify cloning, env preparation, startup), generates SearXNG secret key, stops existing containers. Uses `python-dotenv` (`dotenv_values`).

**Project Name**: All docker-compose commands use `-p localai` (defined in Makefile as `PROJECT_NAME := localai`).

**Version**: Stored in `VERSION` file at repository root.

### Installation Flow

`scripts/install.sh` orchestrates the installation by running numbered scripts in sequence:

1. `01_system_preparation.sh` - System updates, firewall, security hardening
2. `02_install_docker.sh` - Docker and Docker Compose installation
3. `03_generate_secrets.sh` - Generate passwords, API keys, bcrypt hashes
4. `04_wizard.sh` - Interactive service selection (whiptail UI)
5. `05_configure_services.sh` - Service-specific configuration
6. `06_run_services.sh` - Start Docker Compose stack
7. `07_final_report.sh` - Display credentials and URLs
8. `08_fix_permissions.sh` - Fix file ownership for non-root access

The update flow (`scripts/update.sh`) similarly orchestrates: git fetch + reset → service selection → `apply_update.sh` → restart. During updates, `03_generate_secrets.sh --update` adds new variables from `.env.example` without regenerating existing ones (preserves user-set values).

**Git update modes**: Default is `reset` (hard reset to origin). Set `GIT_MODE=merge` in `.env` for fork workflows (merges from upstream instead of hard reset). The `make git-pull` command uses merge mode. Git branch support is explicit: `GIT_SUPPORTED_BRANCHES=("main" "develop")` in `git.sh`; unknown branches warn and fall back to `main`.

## Common Development Commands

### Makefile Commands

```bash
make install           # Full installation (runs scripts/install.sh)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kossakovsky) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
