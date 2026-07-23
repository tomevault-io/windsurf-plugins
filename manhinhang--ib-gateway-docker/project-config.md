---
trigger: always_on
description: This document provides context for AI assistants (like Claude) working with the IB Gateway Docker project.
---

# CLAUDE.md

This document provides context for AI assistants (like Claude) working with the IB Gateway Docker project.

## Project Overview

**IB Gateway Docker** is a lightweight Docker container for Interactive Brokers Gateway (IB Gateway). This project provides a headless, automated solution for running IB Gateway in containerized environments without VNC or GUI dependencies.

### Key Features
- Automated IB Gateway setup and login using IBC (Interactive Brokers Controller)
- Headless operation using Xvfb (virtual framebuffer)
- Health check capabilities (CLI and REST API)
- Minimal attack surface (no VNC, no extra ports)
- Automated version updates via GitHub Actions
- Published to Docker Hub: `manhinhang/ib-gateway-docker`

### Current Versions
- **IB Gateway**: 10.37.1m
- **IBC**: 3.23.0
- **Java**: OpenJDK 17
- **Base Image**: Debian Bookworm Slim

## Project Structure

```
.
├── Dockerfile              # Multi-stage Docker build
├── start.sh               # Container entrypoint script
├── ibc/                   # IBC configuration files
│   └── config.ini        # IBC settings
├── healthcheck/          # Health check tools (Java/Gradle)
│   ├── healthcheck/      # CLI health check tool
│   └── healthcheck-rest/ # REST API health check service
├── test/                 # Python tests using testinfra
│   ├── test_ib_gateway.py
│   ├── test_ib_gateway_fail.py
│   └── test_docker_interactive.py
├── examples/             # Usage examples
│   └── ib_insync/       # ib_insync integration example
├── scripts/              # Utility scripts
│   ├── detect_ibc_ver.py
│   └── extract_ib_gateway_major_minor.sh
├── .github/workflows/    # GitHub Actions CI/CD
│   ├── build-test.yml   # Build and test workflow
│   ├── deploy.yml       # Docker Hub deployment
│   └── detect-new-ver.yml # Automated version detection
└── doc/                 # Documentation assets
```

## Architecture

### Docker Build Stages

The Dockerfile uses a **multi-stage build** approach:

1. **Downloader Stage** (`debian:bookworm-slim`)
   - Downloads IB Gateway installer from Interactive Brokers
   - Downloads latest IBC release from GitHub
   - Extracts version information
   - Prepares IBC configuration

2. **Healthcheck Tools Stage** (`gradle:8.7.0-jdk17`)
   - Builds Java-based health check CLI tool
   - Builds Java-based REST API health check service
   - Creates distribution packages

3. **Final Stage** (`debian:bookworm-slim`)
   - Installs minimal dependencies (Xvfb, Java 17, etc.)
   - Copies IB Gateway, IBC, and health check tools
   - Configures environment and entrypoint

### Runtime Flow

1. `start.sh` is executed as the container entrypoint
2. Xvfb starts on display `$DISPLAY` (headless X server, default `:0`)
3. IBC's `OverrideTwsApiPort` is rewritten to `$IBGW_INTERNAL_PORT`
   (default `4001`); IB Gateway's Java binds that port internally
4. socat forwards external `$IBGW_PORT` (default `4002`) → internal
   `$IBGW_INTERNAL_PORT` when the two ports differ
5. Optional: Health check REST API starts on port 8080
6. IBC launches IB Gateway with provided credentials
7. Cleanup handlers trap INT/TERM signals for graceful shutdown

### Session Persistence

IB Gateway only skips 2FA on launch if it finds an **autorestart file** on
disk (it logs `autorestart file found` vs `autorestart file not found:
full authentication will be required`). That file is *only* written when
IB Gateway performs its own soft restart — never on a normal login, and
never when something kills the JVM externally. The setup uses three
mechanisms together to keep this file fresh:

1. **IBC `AutoRestartTime`** (env var `IBC_AUTO_RESTART_TIME`, default
   `11:00 AM` UTC) schedules an internal JVM soft restart once a day.
   IB Gateway writes the autorestart file then bounces — no 2FA. Primary
   defence against IBKR's ~24h token expiry. The default lands in the gap
   between HK regular close (08:00 UTC) and US regular open (13:30 UTC EDT
   summer / 14:30 UTC EST winter), so the ~60–90s restart window doesn't
   overlap either market's regular session — see *Picking a restart time*
   below if you trade other markets.
2. **IBC command server** (env vars `IBC_COMMAND_SERVER_PORT` default
   `7462`, `IBC_BIND_ADDRESS` default `127.0.0.1`) lets the host send
   `RESTART` over loopback to trigger the same soft-restart codepath on
   demand. Use `./scripts/restart-ib-gateway.sh` to send it.
3. **Persistent `/root/Jts` volume** stores the autorestart file (plus
   `jts.ini` and the device-fingerprint dir) so it survives a container
   exit and a fresh PID 1.

`start.sh` injects all three values into `/root/ibc/config.ini` at boot —
the committed config keeps upstream defaults so the file stays portable.

#### Operational matrix

The autorestart file is **single-use** — IB Gateway writes it at the start
of every soft restart, the next launcher reads it and immediately consumes
it. So the file does *not* sit around waiting for a future restart; only
the soft-restart codepath itself bridges sessions.

| Action | 2FA required? | Use it for |
|--------|---------------|------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [manhinhang/ib-gateway-docker](https://github.com/manhinhang/ib-gateway-docker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
