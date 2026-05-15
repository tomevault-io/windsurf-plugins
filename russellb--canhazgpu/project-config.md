---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is `canhazgpu`, a GPU reservation tool for single host shared development systems. It's a Go CLI application that uses Redis as a backend to coordinate GPU allocations across multiple users and processes, with comprehensive validation to detect and prevent unreserved GPU usage. The system supports both NVIDIA and AMD GPUs through a unified provider abstraction.

## Architecture

The tool is a Go application structured as a CLI with internal packages that implements eight main commands:
- `admin`: Initialize and configure the GPU pool with optional --force flag and --provider selection
- `status`: Show current GPU allocation status with automatic provider-specific validation
- `run`: Reserve GPU(s) and execute a command with `CUDA_VISIBLE_DEVICES` set (blocks if GPUs unavailable)
- `reserve`: Manually reserve GPU(s) for a specified duration (blocks if GPUs unavailable)
- `release`: Release all manually reserved GPUs for the current user
- `report`: Generate GPU reservation reports showing historical reservation patterns by user
- `queue`: Show the GPU reservation queue with wait times and allocation progress
- `web`: Start a web server providing a dashboard for real-time monitoring and reports

### Core Components

- **Redis Integration**: Uses Redis (localhost:6379) for persistent state management with keys under `canhazgpu:` prefix
- **GPU Provider System**: Unified abstraction supporting both NVIDIA (nvidia-smi) and AMD (amd-smi) GPUs
- **GPU Allocation Logic**: Tracks GPU state with JSON objects containing user, timestamps, heartbeat data, and reservation types
- **Heartbeat System**: Background goroutine sends periodic heartbeats (60s interval) to maintain run-type reservations
- **Auto-cleanup**: GPUs are automatically released when heartbeat expires (15 min timeout), manual reservations expire, or processes terminate
- **Unreserved Usage Detection**: Provider-specific integration detects GPUs in use without proper reservations
- **User Accountability**: Process ownership detection identifies which users are running unreserved processes
- **MRU-per-User Allocation**: Most Recently Used per user strategy provides GPU affinity with LRU fallback for fair distribution
- **Specific GPU Reservation**: Users can reserve exact GPU IDs (e.g., --gpu-ids 1,3) when specific hardware is needed
- **Race Condition Protection**: Redis-based distributed locking prevents allocation conflicts
- **Fair Queueing System**: FCFS (First Come First Served) queue with greedy partial allocation for first-in-queue, heartbeat-based stale entry cleanup, and queue status monitoring

## Development Commands

### Build and Installation

```bash
# Option 1: Install directly from GitHub (recommended for users)
go install github.com/russellb/canhazgpu@latest

# Option 2: Build from source
make build            # Build the Go binary to ./build/canhazgpu
make install          # Build and install to /usr/local/bin with bash completion
make test-short       # Run Go tests

# Option 3: Install from local source
go install .          # Installs to $GOPATH/bin or $HOME/go/bin

# Optional: Create short alias symlink (after installing to /usr/local/bin)
sudo ln -s /usr/local/bin/canhazgpu /usr/local/bin/chg
```

### Usage Examples
```bash
# Initialize GPU pool (auto-detects provider)
./build/canhazgpu admin --gpus 8

# Initialize with specific provider
./build/canhazgpu admin --gpus 4 --provider nvidia
./build/canhazgpu admin --gpus 2 --provider amd

# Force reinitialize with different count
./build/canhazgpu admin --gpus 4 --force

# Check status with automatic provider-specific validation
./build/canhazgpu status

# Get JSON output for programmatic use
./build/canhazgpu status --json

# Run command with GPU reservation (by count)
./build/canhazgpu run --gpus 1 -- python train.py

# Run command with specific GPU IDs
./build/canhazgpu run --gpu-ids 1,3 -- python train.py

# Run command with timeout to prevent runaway processes
./build/canhazgpu run --gpus 1 --timeout 2h -- python train.py

# Manual GPU reservation (by count)
./build/canhazgpu reserve --gpus 2 --duration 4h

# Manual reservation of specific GPU IDs
./build/canhazgpu reserve --gpu-ids 0,2,4 --duration 2h

# Release manual reservations
./build/canhazgpu release

# Generate reservation report for last 7 days
./build/canhazgpu report --days 7

# Customize memory threshold for GPU usage detection (default: 1024 MB)
./build/canhazgpu status --memory-threshold 512
./build/canhazgpu run --memory-threshold 2048 --gpus 1 -- python train.py

# Use a configuration file
./build/canhazgpu --config /path/to/config.yaml status
./build/canhazgpu --config config.json run --gpus 2 -- python train.py

# Queueing: wait for GPUs if unavailable (default behavior)
./build/canhazgpu run --gpus 4 -- python train.py  # Waits in queue until GPUs available

# Fail immediately if GPUs are unavailable (no queueing)
./build/canhazgpu run --nonblock --gpus 4 -- python train.py

# Wait up to 30 minutes for GPUs, then fail
./build/canhazgpu run --wait 30m --gpus 4 -- python train.py

# Check the reservation queue

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [russellb/canhazgpu](https://github.com/russellb/canhazgpu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
