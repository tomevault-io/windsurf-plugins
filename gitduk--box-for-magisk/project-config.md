---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**SingBox for Magisk** is a Magisk/KernelSU/APatch module providing transparent proxy functionality for Android devices using sing-box. The module supports three network modes (tproxy, redirect, tun) and manages iptables rules, policy routing, and process lifecycle for the sing-box proxy.

## Architecture

### Module Structure

This is a Magisk module with the following architecture:

- **Root-level scripts**: Module lifecycle hooks (`customize.sh`, `service.sh`, `action.sh`, `build.sh`)
- **`singbox/` directory**: Deployed to `/data/adb/singbox` on device, containing:
  - `bin/`: sing-box and jq binaries
  - `scripts/`: Core service management scripts (modular shell architecture)
  - `config.json`: User's sing-box configuration
  - `settings.ini`: Module settings (network_mode, ipv6, ap_list)
  - `include.list` / `exclude.list`: Application filtering (whitelist/blacklist)
  - `logs/`: Runtime logs with automatic rotation
  - `ui/`: Web UI assets for sing-box dashboard

### Script Architecture (Modular Design)

The codebase follows a **modular shell script architecture** with clear separation of concerns:

**Core Dependencies (source order matters):**
```
constants.sh         # All constants and paths (must load first)
    ↓
utils.sh            # Utility functions (depends on constants)
    ↓
config.sh           # Configuration loading (depends on constants + utils)
    ↓
service.sh          # Main service orchestration
iptables.sh         # Network rules management
```

**Key Scripts:**

1. **`constants.sh`**: Centralized constants definition
   - Paths, ports, timeouts, network constants
   - Intranet IPv4/IPv6 ranges
   - Never modify these directly; they define the entire system's contract

2. **`utils.sh`**: Reusable utility functions
   - Logging with color-coded levels (info/warn/error/debug)
   - Process management (check_process_running, safe_kill, force_kill)
   - Log rotation (automatic when files exceed LOG_MAX_SIZE)
   - File operations, validation helpers

3. **`config.sh`**: Configuration loading and validation
   - Loads `config.json` using jq to extract ports/inbound config
   - Loads `settings.ini` for user preferences
   - Validates configuration and provides friendly error messages
   - Builds intranet address lists including FakeIP ranges

4. **`service.sh`**: Main service orchestration
   - Commands: start, stop, restart, force-stop, status, health
   - Startup sequence: cleanup → validate config → start process → setup iptables → setup IPv6
   - Health checks: process status, config integrity, log errors, network connectivity

5. **`iptables.sh`**: Network rules management
   - Implements three modes: redirect (TCP only), tproxy (TCP+UDP), tun (virtual interface)
   - Creates custom chains: BOX_EXTERNAL, BOX_LOCAL, BOX_IP_V4, BOX_IP_V6
   - Handles application filtering (include/exclude lists via UIDs)
   - Policy routing setup for tproxy mode (fwmark-based routing)
   - IPv6 support with parallel rule structures

6. **`diagnose.sh`**: System diagnostics (not shown but referenced)
7. **`rmlimit.sh`**: Removes vendor network restrictions (called during cleanup)

### Configuration Validation Strategy

The module has **strict configuration validation** with user-friendly error messages:

- `network_mode` in `settings.ini` must match inbound type in `config.json`
- If mismatch detected, shows example configuration with detailed suggestions
- Validates ports, checks TUN device availability, falls back gracefully (TUN → tproxy)
- Config errors include context and actionable fixes, not just "failed"

### Critical Implementation Details

**Process Management:**
- TUN mode runs with full root privileges (needs `CAP_NET_ADMIN` capability)
- Other modes (tproxy/redirect) use `busybox setuidgid root:net_admin` for security
- Wait loops with `MAX_RETRIES` and `RETRY_INTERVAL` for process startup verification
- Graceful shutdown (SIGTERM) with fallback to force kill (SIGKILL) after timeout
- Always cleanup iptables rules before start to ensure clean state

**iptables Architecture:**
- Custom chains avoid polluting system chains
- Order matters: check established connections first (optimization)
- Bypass intranet traffic early to avoid routing loops
- Application filtering by UID (Android's package → UID mapping)
- Cleanup must remove chain references from main chains before deleting chains

**Logging System:**
- Dual output: stdout (colorized for TTY) + log files
- Automatic log rotation at 10MB with 3 backups
- Updates `module.prop` description in real-time for Magisk Manager visibility
- Log levels: info (green), warn (yellow), error (red), debug (cyan)

**IPv6 Handling:**
- Completely separate rule set (ip6tables) paralleling IPv4
- Policy routing with IPv6 fwmark and route table 2024
- Can be disabled via `ipv6="false"` in `settings.ini`
- Adds unreachable rule when IPv6 disabled to prevent leaks

**Application Filtering Priority:**
- `exclude.list` > `include.list` (exclude takes precedence)
- Also reads config.json's `exclude_package[]` / `include_package[]` arrays
- Merged and deduplicated package lists

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gitduk/box_for_magisk](https://github.com/gitduk/box_for_magisk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
