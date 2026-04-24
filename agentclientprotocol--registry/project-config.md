---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Build Commands

```bash
# Build registry (validates all agents and outputs to dist/)
uv run --with jsonschema .github/workflows/build_registry.py

# Dry run (validate without writing to dist/)
uv run --with jsonschema .github/workflows/build_registry.py --dry-run

# Build without schema validation (if jsonschema not available)
python .github/workflows/build_registry.py
```

## Testing & Linting

```bash
# Run workflow tests in the Dockerized local environment (recommended)
.github/workflows/scripts/run-workflows-tests.sh

# Run workflow tests natively on the host (CI-style debugging only)
cd .github/workflows && uv run --with pytest pytest tests/ -v

# Lint check
cd .github/workflows && uv run --with ruff ruff check .

# Format check
cd .github/workflows && uv run --with ruff ruff format --check .

# Auto-fix formatting
cd .github/workflows && uv run --with ruff ruff format .
```

## Docker Validation

GitHub Actions runs the nightly protocol matrix natively on the runner (`uv` + Node.js installed in the workflow), but local verification can still use Docker to keep downloads, caches, and auth-related state isolated from your host machine:

```bash
# Run workflow tests in the Dockerized local environment
.github/workflows/scripts/run-workflows-tests.sh

# Validate schema/build output in a container
.github/workflows/scripts/run-registry-docker.sh uv run --with jsonschema .github/workflows/build_registry.py

# Verify registered agents in a container
.github/workflows/scripts/run-registry-docker.sh python3 .github/workflows/verify_agents.py --auth-check

# Generate the protocol feature matrix in a container
.github/workflows/scripts/run-protocol-matrix.sh

# Generate a capabilities-only matrix table in a container
.github/workflows/scripts/run-protocol-matrix.sh --table-mode capabilities

# Reuse unchanged agent versions from the previous snapshot
.github/workflows/scripts/run-protocol-matrix.sh --table-mode capabilities --changed-only
```

`run-protocol-matrix.sh` mirrors the scheduled GitHub Actions defaults: it uses `--table-mode capabilities` and creates ephemeral isolated Docker state plus a fresh protocol sandbox by default so agents cannot reuse local login/keychain state or stale install artifacts across runs. Set `ACP_PROTOCOL_MATRIX_KEEP_STATE=1` if you intentionally want to keep the container state and `.matrix-sandbox` for debugging.

The generic Docker wrapper keeps state under `.docker-state/` in the repo, defaults to `linux/amd64` to match `ubuntu-latest`, injects a passwd entry for the current UID inside the container, and disables Python keyring backends to avoid host keychain prompts during local verification. It reuses an existing local Docker image when the requested platform and image inputs still match, and rebuilds automatically when they do not; set `ACP_REGISTRY_BUILD_IMAGE=1` to force a rebuild. Treat the Docker wrappers as the canonical local path; use host-native `uv run ...` commands only when you intentionally want to debug outside the container. Set `ACP_REGISTRY_DOCKER_PLATFORM=` to opt out of the default platform override when you explicitly want native-architecture debugging.
Set `ACP_PROTOCOL_MATRIX_SKIP_AGENTS=crow-cli` to skip specific agents during matrix generation.

## Architecture

This is a registry of ACP (Agent Client Protocol) agents. The structure is:

```
<id>/
├── agent.json      # Agent metadata and distribution info
└── icon.svg        # Icon: 16x16 SVG, monochrome with currentColor (required)
```

**Build process** (`.github/workflows/build_registry.py`):

1. Scans directories for `agent.json` files
2. Validates against `agent.schema.json` (JSON Schema)
3. Validates icons (16x16 SVG, monochrome with `currentColor`)
4. Aggregates into `dist/registry.json`
5. Copies icons to `dist/<id>.svg`

**CI/CD** (`.github/workflows/build-registry.yml`):

- PRs: Runs validation only
- Push to main: Validates, then publishes versioned + `latest` GitHub releases

## Validation Rules

- `id`: lowercase, hyphens only, must match directory name
- `version`: semantic versioning (e.g., `1.0.0`)
- `distribution`: at least one of `binary`, `npx`, `uvx`
- `binary` distribution: builds for all operating systems (darwin, linux, windows) are recommended; missing OS families produce a warning
- `binary` archives must use supported formats (`.zip`, `.tar.gz`, `.tgz`, `.tar.bz2`, `.tbz2`, or raw binaries); installer formats (`.dmg`, `.pkg`, `.deb`, `.rpm`, `.msi`, `.appimage`) are rejected
- `icon.svg`: must be SVG format, 16x16, monochrome using `currentColor` (enables theming)
- **URL validation**: All distribution URLs must be accessible (binary archives, npm/PyPI packages)

Set `SKIP_URL_VALIDATION=1` to bypass URL checks during local development.

## Updating Agent Versions

### Automated Updates

Agent versions are automatically updated via `.github/workflows/update-versions.yml`:

- **Schedule:** Runs hourly (cron: `0 * * * *`)
- **Scope:** Checks all agents in the root directory
- **Supported distributions:** `npx` (npm), `uvx` (PyPI), `binary` (GitHub releases only — non-GitHub `repository` URLs are skipped)

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentclientprotocol/registry](https://github.com/agentclientprotocol/registry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
