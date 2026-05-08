---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a **single-command bootstrap script** for deploying a production-ready, highly-available LLM inference stack on AMD ROCm GPUs. The system provides an OpenAI-compatible API endpoint with automatic failover, load balancing, and comprehensive system tuning.

**Key Architecture:**
- Multiple vLLM backend instances (each bound to a subset of GPUs via tensor parallelism)
- HAProxy load balancer in front providing failover, health checking, and sticky sessions
- Containerized deployment via Docker Compose with host networking
- Comprehensive system tuning (kernel, GPU, networking, CPU governor)

The design goal is **redundancy**: clients connect to a single HAProxy endpoint, and individual backend or GPU failures don't cause downtime.

## System Requirements and Compatibility

### Container Runtime: Docker vs Podman

**CRITICAL**: This script requires a **compose tool** to be installed. The script uses `docker compose` command, which requires either:

1. **Docker with Compose V2** (recommended by script author):
   - Real Docker Engine from `get.docker.com`
   - Built-in `docker compose` subcommand

2. **Podman with podman-compose** (common on modern Ubuntu):
   - Podman acts as Docker replacement
   - Ubuntu 25.10+ often has `podman-docker` package that creates `/usr/bin/docker` as a Podman wrapper
   - **REQUIRES** `podman-compose` to be installed separately: `sudo apt-get install podman-compose`
   - Script will fail with "looking up compose provider failed" error if compose tool is missing

**How to detect your setup:**
```bash
# Check if docker is really Podman
ls -la /usr/bin/docker  # Small file (~228 bytes) = Podman wrapper

# Check your container runtime
docker --version  # Shows "podman version" if using Podman

# Verify compose tool availability
docker compose version    # Docker Compose V2
podman-compose --version  # Podman Compose
```

**If script fails with compose errors:**
```bash
# For Podman users (Ubuntu 25.10+):
sudo apt-get install -y podman-compose

# For Docker users:
sudo apt-get install -y docker-compose-v2

# Or install real Docker (removes Podman):
sudo apt-get remove -y podman-docker
curl -fsSL https://get.docker.com | sudo sh
```

### OS Version Compatibility

- **Designed for**: Ubuntu 24.04 LTS
- **Tested on**: Ubuntu 25.10 (Questing Quokka) with Podman
- **Warning shown**: Script detects non-24.04 and shows warning but continues

### GPU Configuration Notes

The default configuration assumes **4 GPUs**:
- `EXPECTED_GPU_COUNT=4` in `launch.sh`
- `VLLM1_GPUS="0,1"` and `VLLM2_GPUS="2,3"` (2 backends × TP=2)

**If you have a different GPU count**, update `launch.sh`:
```bash
# For 6 GPUs (example):
export EXPECTED_GPU_COUNT="6"
export VLLM1_GPUS="0,1,2"    # Backend 1: GPUs 0,1,2 with TP=3
export VLLM2_GPUS="3,4,5"    # Backend 2: GPUs 3,4,5 with TP=3

# Or for 2 GPUs:
export EXPECTED_GPU_COUNT="2"
export VLLM1_GPUS="0"        # Backend 1: GPU 0 with TP=1
export VLLM2_GPUS="1"        # Backend 2: GPU 1 with TP=1
```

## Core Scripts

### haproxllm.sh (Main Installation Script)
The primary installation and deployment script (~1500 lines). This is a **complete system bootstrap** that:
- Installs ROCm 6.3 and AMDGPU drivers (if not present)
- Configures Docker for ROCm GPU access
- Downloads HuggingFace models
- Auto-detects GPU type and selects optimized vLLM container images
- Generates HAProxy configuration with health checks and sticky sessions
- Creates Docker Compose stack with two vLLM backends
- Applies production system tuning (sysctl, ulimits, GPU power profiles, CPU governor)
- Optionally generates TLS certificates for HTTPS with HTTP/2

**Important:** This script is designed to be **run once** to set up the stack. It is **idempotent** for most operations (skips installation if already present).

### launch.sh (Configuration Wrapper)
A user-friendly configuration wrapper that sets environment variables and then executes `haproxllm.sh`. Users should modify this file to customize their deployment instead of editing the main script.

## Running the Stack

### Initial Setup
```bash
# Configure your settings in launch.sh, then run:
sudo bash launch.sh

# Or run directly with environment overrides:
sudo MODEL_ID="openai/gpt-oss-20b" GPU_TYPE="r9700" bash haproxllm.sh
```

### Managing the Stack
```bash
# Note: Use 'podman-compose' if using Podman, or 'docker compose' for Docker

# View logs
docker compose -f /opt/llm-stack/compose/docker-compose.yml logs -f
# OR for Podman:
podman-compose -f /opt/llm-stack/compose/docker-compose.yml logs -f

# Restart services
docker compose -f /opt/llm-stack/compose/docker-compose.yml restart

# Stop services
docker compose -f /opt/llm-stack/compose/docker-compose.yml down

# Start services
docker compose -f /opt/llm-stack/compose/docker-compose.yml up -d

# Check GPU status
rocm-smi
rocm-smi --showmeminfo vram
watch -n 1 rocm-smi  # Continuous monitoring

# View HAProxy stats
curl http://wide.local:8404/stats

# Check individual backend health
curl http://wide.local:8001/health  # vllm1
curl http://wide.local:8002/health  # vllm2
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marked23/HAProxy-vLLM-bootstrap](https://github.com/marked23/HAProxy-vLLM-bootstrap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
