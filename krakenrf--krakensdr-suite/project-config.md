---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains **Heimdall v2**, a real-time direction-finding (DoA) system for RTL-SDR devices. The project consists of two main C++ applications that work together:

1. **Heimdall Server** (`heimdall_v2/`): Multi-RTL-SDR coherent receiver with phase compensation
2. **DoA Client** (`kraken_doa_v2/`): FFT viewer and MUSIC DoA processor

Both applications are optimized for ARM platforms (Raspberry Pi 4/5) with NEON SIMD support, but also run on x86_64.

## Repository Structure

```
krakensdr_v2/                        # repository root
├── heimdall_v2/                     # Main server application
│   ├── src/
│   │   ├── core/       # Core types, config, settings, logging, utilities
│   │   ├── sdr/        # RTL-SDR device management and sample pipeline
│   │   ├── dsp/        # FFT, correlation, compensation algorithms
│   │   ├── net/        # TCP servers for data/control/RTL-TCP
│   │   ├── web/        # uWebSockets web interface
│   │   └── main.cpp    # Server entry point
│   ├── external/concurrentqueue/    # Vendored moodycamel queue
│   ├── config.h        # Main configuration file
│   ├── Makefile        # Primary build system
│   └── index.html      # Web UI
│
└── kraken_doa_v2/                   # DoA client application
    ├── src/
    │   ├── signal_processing/  # FFT, FM demod, MUSIC DoA, beamformer, decimator
    │   ├── networking/         # TCP client, data receiver, WebSocket server
    │   ├── utils/              # Ring buffers, IQ conversion, stats
    │   ├── channel_manager.cpp
    │   ├── scanner_manager.cpp
    │   └── main.cpp            # Client entry point
    ├── include/
    │   ├── config.hpp          # Client configuration
    │   └── globals.hpp         # Global state declarations
    ├── Makefile                # Client build system
    └── kraken_doa.html         # Client web UI
```

## Build Commands

### Heimdall Server

```bash
cd heimdall_v2

# Quick start
make              # Build (auto-installs uWebSockets)
make run          # Build and run server
make deps         # Install system dependencies (Ubuntu/Debian)

# Alternative distributions
make deps-fedora  # Fedora/RHEL
make deps-arch    # Arch Linux

# Build management
make clean        # Remove build artifacts
make rebuild      # Clean and rebuild
make distclean    # Remove everything including uWebSockets
make status       # Check dependencies and module status
make debug        # Show build configuration

# CMake alternative (needs ../librtlsdr/build/src/librtlsdr.a - built by
# install.sh or by `make` above; CMake stops with a message if it's missing)
mkdir build && cd build
cmake ..
make -j3          # never more than 3 jobs on the Pi (see kraken_doa_v2/CLAUDE.md)
./heimdall
```

### DoA Client

```bash
cd kraken_doa_v2

# Quick start
make              # Build (auto-installs uWebSockets)
make run          # Build and run client
make deps         # Install system dependencies

# Build management
make clean        # Remove build artifacts
make rebuild      # Clean and rebuild
make distclean    # Remove everything including uWebSockets
make status       # Check dependencies and build status

# Debugging
make debug        # Build with debug symbols
make debug-arm    # ARM build with NEON debug output
```

## Running the Stack (install.sh / run.sh)

- `./install.sh` installs apt dependencies, builds the librtlsdr fork (see
  *Dependencies*) and both apps. It does NOT remove distro RTL-SDR packages -
  heimdall links the fork statically, so gqrx, gr-osmosdr, rtl_433 etc. can
  stay installed on the stock library.
- `./run.sh [--wideband|-w] [--kerberos] [--kerberos_sw|--kerberos-sw]` starts
  both apps in a tmux split (`NO_TMUX=1` = headless, logs in `logs/`);
  `./run.sh stop` stops everything. Unknown arguments are an error (a
  mistyped flag used to be ignored silently); `-h` prints usage.
- Each app runs under a supervisor (`run.sh __supervise`) that restarts it
  after a crash. Ctrl+C, `run.sh stop`, `systemctl stop` and closing the tmux
  pane/window all stop it cleanly: the supervisor runs the app as a
  background child it `wait`s on and forwards the signal, because a pane close
  SIGHUPs only the supervisor (the pane's session leader), never the app.
- Starting AND `stop` sweep stale processes first: any heimdall / kraken_doa /
  run.sh supervisor outside the current session (a dead tmux server, an old
  run, a headless run) gets SIGTERM per process group (clean shutdown,
  dongles released), SIGKILL after 10 s. Processes of other users are only
  reported.
- The convergence wait before the client starts watches heimdall's process:
  headless mode aborts with heimdall's log tail if it dies; in tmux mode the
  client pane gives up if the heimdall pane's supervisor exits (a crash is
  restarted by the supervisor, so the wait continues through it).
- Headless mode has no supervisor: when heimdall ends on its own, run.sh
  exits with heimdall's status (1 = startup failure, 128+N = killed by signal
  N, e.g. 139 for a crash); a stop it was asked for (Ctrl+C / TERM) exits 0.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [krakenrf/krakensdr_suite](https://github.com/krakenrf/krakensdr_suite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
