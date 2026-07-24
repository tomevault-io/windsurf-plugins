---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

fivenines-agent is a monitoring agent that collects server metrics and sends them to the fivenines API (https://fivenines.io). The agent runs continuously, probing system capabilities and collecting various metrics at configurable intervals.

## Development Commands

### Setup
```bash
# Install dependencies using Poetry
make install
```

### Code Quality
```bash
# Run linters (isort, black, flake8, mypy, bandit)
make lint

# Auto-format code (isort, black)
make format

# Run tests with coverage (requires 100% coverage)
make test

# Run a single test file
poetry run pytest tests/test_collectors.py -v
```

### Build Binary
```bash
# Build standalone executable for Linux (uses PyInstaller)
./py2exe.sh
```

The build process creates a self-contained binary at `./dist/linux/fivenines-agent-linux-*/` that includes all dependencies (libvirt, libcrypt, etc.) for compatibility with CentOS 7+.

### Running the Agent
```bash
# Run directly with Poetry
poetry run fivenines_agent

# Run with dry-run mode (collects metrics once and exits, prints JSON to stdout)
poetry run fivenines_agent --dry-run

# Check version
poetry run fivenines_agent --version
```

## Architecture

### Core Components

**Agent Loop (`agent.py`)**
- Main orchestrator that runs the collection loop
- Handles signals (SIGTERM, SIGINT for shutdown; SIGHUP for capability refresh)
- Collects static info once (version, uname, boot time, capabilities, user context)
- On each iteration: collects configured metrics, enqueues data, sleeps until next interval
- Manages graceful shutdown with proper cleanup

**Permission Probing (`permissions.py`)**
- Detects available monitoring capabilities at startup based on file permissions, sudo access, and group memberships
- Re-probes automatically every 5 minutes or on SIGHUP signal to detect permission changes
- Capabilities include: core metrics (always available), hardware sensors, storage (SMART/RAID), services (Docker/QEMU/Proxmox/systemd), kernel surfaces (cgroup, tri-state "v1"/"v2"/None), security (fail2ban), logs (`journald` read access, probed via `journalctl -n 0`), etc.
- Prints a capabilities banner showing what features are available/unavailable with hints

**Synchronizer (`synchronizer.py`)**
- Background thread that sends collected data to the fivenines API
- Fetches configuration from server before starting metric collection
- Handles retries with exponential backoff
- Compresses data with gzip before sending
- Uses custom DNS resolution with IPv4/IPv6 fallback

**SynchronizationQueue (`synchronization_queue.py`)**
- Thread-safe queue with maxsize limit for buffering collected metrics
- Prevents memory exhaustion if API is unreachable

**Log Uploader (`log_uploader.py`, `log_capture.py`)**
- Dedicated `LogUploader` worker thread + bounded queue that upload incident log-capture bundles to `/logs` via `Synchronizer.send_logs`, kept off the metric-collection loop so a slow/large upload never stalls collection, `/collect`, or the systemd watchdog
- `CaptureCoordinator` applies the backend `capture_logs` command with a capture_id nonce + on-disk `last_capture_id` persistence: each command fires exactly once and never replays after a `Restart=always` restart; `last_served` advances only after a confirmed upload, so a failed capture retries
- Part of log-monitoring V1; inert until the backend implements the `/collect` `capture_logs` command and the `/logs` endpoint

**Subprocess Utilities (`subprocess_utils.py`)**
- Critical for PyInstaller compatibility: removes LD_LIBRARY_PATH and other environment variables that can interfere with system commands
- PyInstaller bundles libraries (like libselinux from libvirt) that conflict with system utilities (sudo, smartctl, mdadm)
- Always use `get_clean_env()` when calling subprocess commands

**Environment (`env.py`)**
- Central source for runtime configuration: `api_url()`, `config_dir()`, `dry_run()`, `log_level()`
- Config directory defaults to `/etc/fivenines_agent`; override with `CONFIG_DIR` env var
- `get_user_context()` collects user/group info sent with each payload

**Collector Registry (`collectors.py`)**
- Declarative `COLLECTORS` list maps config keys to `(data_key, callable, pass_kwargs)` tuples
- `agent.py` iterates this registry each tick; `pass_kwargs=True` unpacks the config dict as `**kwargs` to the callable
- Add new metrics here rather than modifying the agent loop

### Metric Collectors

Each metric collector is a separate module that exports functions to collect specific metrics:

- **Core metrics** (always enabled): `cpu.py`, `memory.py`, `load_average.py`, `io.py`, `network.py` (per-interface byte/packet/error/drop counters, plus Linux bridge detection, `interface_type` bridge/physical/virtual, `network_link_speed_bps` from `/sys/class/net/<if>/speed`, and `bridge_member_count` so the backend can compute per-interface saturation), `partitions.py`, `files.py`, `ports.py`, `processes.py`, `temperatures.py`, `fans.py`
- **Storage**: `smart_storage.py` (requires sudo smartctl), `raid_storage.py` (requires sudo mdadm), `zfs.py`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Five-Nines-io/fivenines_agent](https://github.com/Five-Nines-io/fivenines_agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
