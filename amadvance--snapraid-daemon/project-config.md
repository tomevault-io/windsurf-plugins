---
trigger: always_on
description: SnapRAID-Daemon is a specialized companion service that transforms SnapRAID from a manual command-line interface (CLI) workflow into an 'always-on' background service. It provides automated maintenance, health monitoring, power management, and a REST API for remote control and monitoring.
---

# SnapRAID Daemon Project Documentation

## Overview

SnapRAID-Daemon is a specialized companion service that transforms SnapRAID from a manual command-line interface (CLI) workflow into an 'always-on' background service. It provides automated maintenance, health monitoring, power management, and a REST API for remote control and monitoring.

The daemon doesn't reimplement SnapRAID's core functionality. Instead, it manages and orchestrates the existing SnapRAID CLI binary, providing the same level of reliability with enhanced automation and monitoring capabilities.

### Key Features

- **Automated Maintenance**: Scheduled sync and scrub operations with configurable thresholds
- **SMART Monitoring**: Continuous disk health monitoring with predictive failure analysis
- **Power Management**: Automatic disk spindown during inactivity
- **Task Queue**: Sequential task execution with progress tracking and logging
- **REST API**: HTTP interface for monitoring and control
- **Web Server**: Embedded CivetWeb serving static dashboard files
- **Notification System**: Syslog, webhooks, and custom script hooks
- **State Persistence**: Task history and logs survive daemon restarts

##Project Structure

### Core Source Files (`daemon/`)

The daemon is organized into focused C modules:

| Module | Purpose |
|--------|---------|
| `daemon.c` | Main entry point, daemonization, signal handling |
| `state.c/h` | Global state management, threading primitives (mutex/rwlock) |
| `runner.c/h` | Task execution engine, spawns SnapRAID processes, parses output |
| `scheduler.c/h` | Cron-like scheduling thread for automated maintenance |
| `parser.c/h` | Parses SnapRAID CLI output to extract progress, errors, SMART data |
| `rest.c/h` | REST API endpoints implementation (OpenAPI 3.1.0 compliant) |
| `web.c/h` | Static file serving, MIME types, compression support |
| `conf.c/h` | Configuration file parser and runtime config PATCH support |
| `log.c/h` | Logging subsystem (syslog integration) |
| `elem.c/h` | Memory management for data structures (disks, tasks, messages) |
| `report.c/h` | Text report generation for notifications |
| `support.c/h` | Utility functions (string handling, SMART analysis, disk probing) |
| `unix.c` | UNIX-specific system calls (fork, exec, user switching) |

### State Management (`daemon/state.h`)

The daemon maintains a centralized `struct snapraid_state` protected by thread synchronization primitives:
- A **Mutex** (`thread_mutex_t`) protects the overall state, accessed via `state_lock()` / `state_unlock()`.
- A **Read-Write Lock** (`thread_rwlock_t`) protects the in-memory web asset cache (`page_list`), allowing concurrent reads while serving static files.
- **Condition Variables** (`thread_cond_t`) are used by the runner and scheduler threads for efficient waiting and signaling.

#### Core State Components

- **`pulse`**: Monotonic counters for cache invalidation (array, config, disks, tasks, activity)
- **`runner`**: Task queue manager with worker thread
- **`scheduler`**: Automated maintenance scheduler with dedicated thread
- **`global`**: Array metadata (sync/scrub timestamps, block counters, diff stats)
- **`config`**: Runtime configuration (network, maintenance, monitoring, notifications)
- **`data_list` / `parity_list`**: Disk inventory with SMART telemetry
- **`page_list`**: In-memory cache of web assets (if `net_web_root` is configured)

#### Threading Model

The daemon uses a multi-threaded architecture:

1. **Main Thread**: Handles signals, config reload (SIGHUP), REST API requests
2. **Runner Thread**: Sequential task executor (one task at a time)
3. **Scheduler Thread**: Wakes up periodically to check maintenance schedule
4. **CivetWeb Threads**: HTTP request handlers (thread pool)

All access to shared state is protected by `state_lock()` / `state_unlock()`.

### Task Execution Model (`daemon/runner.h`)

The `runner` subsystem implements a queue-based task manager:

#### Task Lifecycle

```
QUEUE → START → RUN → (SIGNAL|TERM|CANCEL)
```

- **QUEUE**: Task waiting in `waiting_list`
- **START**: Task initializing (loading content file)
- **RUN**: Task processing blocks (progress percentage available)
- **SIGNAL**: Task received SIGTERM
- **TERM**: Task exited with status code
- **CANCEL**: Task skipped due to dependency failure or threshold violation

#### Task Structure (`struct snapraid_task`)

Key fields:
- `cmd`: Command ID (sync, scrub, diff, status, etc.)
- `pid`: Process ID of running SnapRAID instance
- `progress`, `eta_seconds`, `speed_mbs`: Real-time telemetry
- `message_list`: Structured log messages (fatal, error, info)
- `error_io`, `error_data`, `error_soft`: Error counters
-`block_*`: Block processing metrics (begin, end, idx, done)

### API Architecture (`snapraidd.yaml`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amadvance/snapraid-daemon](https://github.com/amadvance/snapraid-daemon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
