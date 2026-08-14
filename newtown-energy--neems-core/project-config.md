---
trigger: always_on
description: This document contains important instructions for AI agents (like Claude Code) working on the NEEMS Core project.
---

# AI Agent Instructions for NEEMS Core

This document contains important instructions for AI agents (like Claude Code) working on the NEEMS Core project.

**IMPORTANT: This file should be regularly updated as you learn new patterns, workflows, or project-specific conventions. When you discover something important about how this project works, update this file to capture that knowledge for future sessions.**

## Critical Rules

### Docker Usage

**ALWAYS run commands inside Docker containers, NEVER on the host machine.**

This project uses `../devenv` to coordinate Docker containers. The devenv directory is located one level up from the neems-core directory.

**You MUST use `docker compose exec` from the devenv directory, NOT `docker exec`.**

Key points:
- All commands must be run via `docker compose exec` from `/Users/slifty/Maestral/Code/open-tech-strategies/newtown/devenv`
- Use `cd /Users/slifty/Maestral/Code/open-tech-strategies/newtown/devenv && docker compose exec neems-api <command>`
- The Docker Compose configuration is in `../devenv/docker-compose.yml`
- Never run cargo, tests, or build commands directly on the host
- Check container status with `cd /Users/slifty/Maestral/Code/open-tech-strategies/newtown/devenv && docker compose ps`

**Examples:**

✅ **Correct:**
```bash
cd /Users/slifty/Maestral/Code/open-tech-strategies/newtown/devenv && docker compose exec neems-api /usr/src/app/bin/dosh lint-clippy
cd /Users/slifty/Maestral/Code/open-tech-strategies/newtown/devenv && docker compose exec neems-api /usr/src/app/bin/dosh test
cd /Users/slifty/Maestral/Code/open-tech-strategies/newtown/devenv && docker compose exec neems-api cargo build
```

❌ **Incorrect:**
```bash
docker exec neems-api /usr/src/app/bin/dosh lint-clippy  # Wrong - use docker compose exec
./bin/dosh lint-clippy  # Wrong - DO NOT run on host
cargo test              # Wrong - DO NOT run on host
cargo build             # Wrong - DO NOT run on host
cargo fmt               # Wrong - DO NOT run on host
```

### Docker Tooling Setup

The Docker containers include all necessary Rust development tools:
- `clippy` - for linting
- `rustfmt` - for code formatting
- `rust-src` - for enhanced IDE support and development

These components are installed via `rustup component add clippy rustfmt rust-src` in the Dockerfiles for both `neems-api` and `neems-data` services.

If you need to rebuild the containers to pick up Dockerfile changes:
```bash
cd /Users/slifty/Maestral/Code/open-tech-strategies/newtown/devenv && docker compose build neems-api neems-data
cd /Users/slifty/Maestral/Code/open-tech-strategies/newtown/devenv && docker compose up -d neems-api neems-data
```

## Project Structure

This is a Rust workspace with multiple crates:
- `neems-api` - Main API server
- `neems-admin` - CLI administration tool
- `neems-data` - Data aggregation service (contains the RTAC Modbus integration in `src/rtac/`)
- `neems-rtac-sim` - Simulated RTAC: a Modbus TCP server for exercising the RTAC integration without hardware
- `crates/fixphrase` - Utility crate for GPS coordinate encoding

## Simulated RTAC (`neems-rtac-sim`)

`neems-rtac-sim` is a standalone Modbus TCP server that simulates an RTAC. It
reuses the register map from `neems-data`'s `rtac::protocol` (the single source
of truth), so the simulator and the real client cannot drift apart. Command
registers (target charge, operating mode) drive a simple internal model whose
state of charge and alarm list advance once per tick (1 Hz by default).

- Run it interactively: `cargo run -p neems-rtac-sim` (type `help` for stdin
  control commands like `charge`, `discharge`, `soc 80`, `alarm set 321`).
- In `devenv` it runs as the `neems-rtac-sim` service (with `--no-stdin`,
  binding `0.0.0.0:502`). `neems-data` is pointed at it by default via the
  `RTAC_ADDRESS=neems-rtac-sim:502` env var, read by `RtacConfig::from_env()`.

### RTAC collector (`rtac::runner`)

`neems-data monitor` starts the RTAC collector alongside the source poller (see
`DataAggregator::start_aggregation`). `rtac::runner::run_rtac_collector` wires
the `ModbusWorker` to the storage and alarm tasks, polls the RTAC at 10 Hz, and
persists SoC readings at 1 Hz to a dedicated `charging_state` source named
`rtac` (created `active = false` so the generic poller doesn't also write it).
Those readings surface through the existing `GET /Sites/<id>/SocHistory`
endpoint and the React dashboard.

- The collector is **closed-loop**: alongside reading status it drives the RTAC
  from the site's schedule. A background poller (`rtac::schedule_http`) fetches
  the active command from neems-api's `GET /Sites/<id>/ActiveCommand` endpoint,
  and `ControlLogicTask` (via `HttpScheduleProvider`) turns it into RTAC commands
  with reactive SoC/alarm safety overrides. Credentials come from
  `NEEMS_API_URL` / `NEEMS_API_EMAIL` / `NEEMS_API_PASSWORD` (falling back to
  `NEEMS_DEFAULT_EMAIL` / `NEEMS_DEFAULT_PASSWORD`); without them the collector
  stays read-only and logs a warning.
- It runs on its own thread with a current-thread runtime because the Modbus
  client context is not `Send`.
- Disabled by default so the system doesn't follow a (possibly absent) RTAC

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Newtown-Energy/neems-core](https://github.com/Newtown-Energy/neems-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
