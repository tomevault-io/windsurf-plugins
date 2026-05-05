---
trigger: always_on
description: This document orients contributors and AI agents working on this repo. It captures
---

# AGENTS.md

This document orients contributors and AI agents working on this repo. It captures
the key project concepts and default guidelines. Update as needed.

## Project Summary
- Purpose: Docker image for self-hosting Hytale Dedicated Servers with flexible
  server file management (launcher copy, CLI download, or manual).
- Runtime: Alpine Linux with Eclipse Temurin JRE (Java 25 by default).
- Implementation: TypeScript compiled to standalone Bun binaries (no bash/curl/jq/unzip dependencies).
- Entry: `/opt/hytale/bin/entrypoint` inside the container (compiled binary).
- Security: runs as non-root user (UID/GID 1000).
- Data: persistent `/data` volume.
- Image: Published to `ghcr.io/godstepx/docker-hytale-server`
- Dev Tools: `just` for task automation (see Justfile)

## Core Concepts
- Download modes:
  - `manual`: user provides `/data/server` and `/data/Assets.zip`.
  - `launcher`: copy from `LAUNCHER_PATH`.
  - `cli`: official Hytale Downloader CLI + OAuth device flow.
  - `auto`: try launcher, then CLI, else manual instructions.
- Bundled CLI:
  - Pre-downloaded at build time to `/opt/hytale/cli/` (read-only).
  - Eliminates runtime CLI download - only server files require OAuth.
  - Falls back to `/data/.hytale-cli/` if bundled CLI not found.
- Version tracking: `/data/.version` JSON with source, patchline, timestamp.
- Auth caches:
  - Downloader CLI tokens in `/data/.auth` (persisted in volume).
- AOT cache support: `/data/server/HytaleServer.aot` used if present (Java 25+).
- Health checks:
  - `/opt/hytale/bin/healthcheck` binary checks Java process and UDP port 5520.
- Logs: `/data/logs` (PID file: `/data/server.pid`).

## Repository Layout
- `Dockerfile`: multi-stage build (Bun compilation + CLI download + production), env defaults, healthcheck, non-root user.
- `Justfile`: development task runner (build, test, lint, format, etc.).
- `src/entrypoint.ts`: main entrypoint - downloads files, writes configs, acquires tokens, starts Java, handles signals. Compiled to binary.
- `src/setup.ts`: setup utilities - Java args building, file validation. Imported by entrypoint.
- `src/config-writer.ts`: config.json and whitelist.json generation from env vars. Imported by entrypoint.
- `src/token-manager.ts`: OAuth2 device flow, token refresh, session creation. Imported by entrypoint (not a standalone binary).
- `src/download.ts`: download/copy server files + version tracking - imported by entrypoint.
- `src/mod-installer.ts`: CurseForge mod installer (auto-install mode) - imported by entrypoint.
- `src/config.ts`: centralized configuration with all env var defaults.
- `src/healthcheck.ts`: health checks for Docker - compiled to binary.
- `src/log-utils.ts`: logging module (imported by other TypeScript modules).
- `package.json`: Bun project manifest with build scripts.
- `tsconfig.json`: TypeScript compiler configuration.
- `README.md`: user-facing usage instructions and env vars.
- `tests/test-integration.sh`: integration tests for Docker container.

## Container Runtime Flow
1. Entrypoint binary (`/opt/hytale/bin/entrypoint`) sets up `/data` directories.
2. Config writer generates `config.json` and `whitelist.json` from env vars (if needed).
3. Download module ensures server files are present (by mode: cli, launcher, or manual).
4. Token manager acquires OAuth tokens and creates game session (or uses env vars).
5. Java starts `HytaleServer.jar` with assets, auth tokens, and command-line args.
6. Background OAuth refresh loop keeps tokens alive for indefinite runs (30+ days).
7. SIGTERM -> graceful shutdown (30s timeout); SIGKILL if needed.

**Technical Implementation:**
- TypeScript compiled to standalone Bun binaries during Docker build.
- 2 binaries: `entrypoint` (includes token-manager module) and `healthcheck`.
- Binaries are self-contained (no Node.js/Bun runtime needed in production image).
- Uses Bun's built-in APIs: `fetch()` for HTTP, native JSON parsing, system `unzip` for extraction.
- Process management via `Bun.spawn()` for external commands (Java, Hytale CLI, system utils).
- Internal modules use standard TypeScript imports (bundled, no process spawning).
- OAuth tokens stored in `/data/.auth/.oauth-tokens.json` for persistence across restarts.
- Hytale server handles game session refresh internally when given `--session-token`.

## Server Command-Line Flags (Verified)
All flags documented via `java -jar HytaleServer.jar --help`:
- `--assets <Path>`: Asset directory
- `--bind <InetSocketAddress>`: Address to listen on (default: 0.0.0.0:5520)
- `--auth-mode <authenticated|offline>`: Authentication mode (default: authenticated)
- `--session-token <token>`: Pre-configured session token (JWT)
- `--identity-token <token>`: Pre-configured identity token (JWT)
- `--owner-uuid <uuid>`: Profile UUID for session
- `--backup`: Enable automatic backups
- `--backup-dir <Path>`: Backup directory
- `--backup-frequency <Integer>`: Backup interval in minutes (default: 30)
- `--backup-max-count <Integer>`: Maximum number of backups to keep (default: 5)
- `--allow-op`: Allow operator commands
- `--accept-early-plugins`: Acknowledge loading early plugins (unsupported)
- `--early-plugins <Path>`: Additional early plugin directories
- `--disable-sentry`: Disable crash reporting

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [godstepx/docker-hytale-server](https://github.com/godstepx/docker-hytale-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
