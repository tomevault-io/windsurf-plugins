---
trigger: always_on
description: This document provides comprehensive guidelines for AI agents working on the /dev/push codebase. It covers scripts, FastAPI application code, Docker/Compose, and project-wide conventions.
---

# AI Agent Guidelines for /dev/push

This document provides comprehensive guidelines for AI agents working on the /dev/push codebase. It covers scripts, FastAPI application code, Docker/Compose, and project-wide conventions.

## Table of Contents

- [Scripts (`scripts/`)](#scripts-scripts)
- [FastAPI Application (`app/`)](#fastapi-application-app)
- [Docker & Compose](#docker--compose)
- [Project Structure](#project-structure)
- [General Conventions](#general-conventions)
- [Testing & Deployment](#testing--deployment)

---

## Scripts (`scripts/`)

These guidelines apply to every script under `scripts/` (install/start/stop/restart/helpers/etc.).

### Environment Detection & Paths

1. **Always source `scripts/lib.sh`** (use the `SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"` pattern, then `source "$SCRIPT_DIR/lib.sh"`). The lib sets `APP_DIR`, `DATA_DIR`, `ENV_FILE`, etc., and auto-detects dev vs prod via systemd or `DEVPUSH_ENV`.
2. **Do not hardcode relative paths.** Derive everything from `APP_DIR`, `DATA_DIR`, or `SCRIPT_DIR`.
3. **Expose overrides via `DEVPUSH_*` env vars** (already handled by the lib).

### Docker / Compose Usage

1. Call `ensure_compose_cmd` before issuing any compose commands (or rely on `set_compose_base` which calls it internally).
2. Use `set_compose_base` to populate `COMPOSE_BASE` (reads `CERT_CHALLENGE_PROVIDER` from `.env` internally).
3. Run compose via `run_cmd "Message..." "${COMPOSE_BASE[@]}" <subcommand> …`. Never spell `docker compose` / `docker-compose` directly.
4. `set_compose_base` also ensures `SERVICE_UID`/`SERVICE_GID` are exported so Docker builds run with the correct user. Never assume UID/GID 1000; always go through the helper.

### Output & Spacing

1. **Use `run_cmd` for every non-trivial operation** (package installs, docker commands, helper scripts). It handles spinners, logging, and error capture.
2. At the top of each logical section add a short comment (`# Install Docker`, `# Start stack`, etc.) so the script reads like a TOC. Skip obvious blocks like `usage()`.
3. For blank lines between major blocks, call `printf '\n'` once—no bare `echo`.
4. When printing status messages manually (e.g., final "Success" line), use `printf "${GRN}…${NC}\n"` for consistency.
5. Use parent/child command structure for multi-step operations:
   ```bash
   printf "Installing...\n"
   printf "%s Building runner images...\n" "$CHILD_MARK"
   run_cmd "${CHILD_MARK} Starting services..." "${COMPOSE_BASE[@]}" up -d
   ```

### Flags & CLI UX

1. Keep flag sets minimal; only add options when they're truly needed (e.g., `--no-migrate`, `--timeout <value>`).
2. In usage blocks, show value placeholders as `<value>` and list allowed values inline.
3. Validate flag values early and exit via `usage` on invalid input.
4. For sensitive input (tokens, passwords), make flags optional and prompt securely with `read -s` if not provided and TTY is available.

### Helper Scripts

1. Prefer shared helpers over inline logic:
   - DB migrations: `run_cmd "Running database migrations..." bash "$SCRIPT_DIR/db-migrate.sh"`.
2. If a helper emits output, rely on its own logging (no extra text before/after unless absolutely necessary).

### Comments & Structure

1. Break scripts into clear sections with comments (e.g., `# Create data directories`, `# Validate core env`).
2. Within a section, keep related commands together and avoid interleaving unrelated work.
3. Use `set -Eeuo pipefail` and a trap that prints the last command and `SCRIPT_ERR_LOG` (see `start.sh` / `install.sh` for reference).
4. It's fine to precede the argument-parsing block with a short comment like `# Parse CLI flags` for readability.

### Miscellaneous

1. Avoid `echo` unless you truly need the "no newline" behavior; prefer `printf`.
2. When running commands as the service user from privileged scripts (e.g., install), wrap them in `runuser -u "$user" -- bash -c '…'` so files are owned by `devpush`.
3. When creating files/dirs that might already exist, guard with `[[ ! -f … ]]` / `install -d …` and let them be no-ops if present.
4. For comment documentation aimed at future maintainers, keep it short and factual—no personal notes or TODOs; use `AGENTS.md` instead.
5. Use `validate_env "$ENV_FILE"` whenever you need to enforce required environment variables; it handles core values and certificate-challenge-specific secrets for production.

---

## FastAPI Application (`app/`)

### Project Structure

- `app/routers/`: Route handlers organized by domain (auth, project, team, user, admin, etc.)
- `app/forms/`: WTForms form definitions (one file per domain)
- `app/models.py`: SQLAlchemy ORM models
- `app/services/`: Business logic services (GitHub, deployment, domain, Loki, etc.)
- `app/utils/`: Utility functions (access control, pagination, color, etc.)
- `app/templates/`: Jinja2 templates organized by domain
- `app/workers/`: Background workers (jobs, monitor) and tasks
- `app/config.py`: Pydantic settings and configuration
- `app/dependencies.py`: FastAPI dependencies and template helpers
- `app/db.py`: Database connection and session management

### Routing Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hunvreus/devpush](https://github.com/hunvreus/devpush) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
