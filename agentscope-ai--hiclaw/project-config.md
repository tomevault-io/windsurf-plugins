---
trigger: always_on
description: This file helps AI Agents (and human developers) quickly understand the project structure and find relevant code.
---

# HiClaw Codebase Navigation Guide

This file helps AI Agents (and human developers) quickly understand the project structure and find relevant code.

## What is HiClaw

HiClaw is an open-source Agent Teams system that uses IM (Matrix protocol) for multi-Agent collaboration with human-in-the-loop oversight. It consists of a Manager Agent (coordinator) and Worker Agents (task executors), connected via an AI Gateway (Higress), Matrix Homeserver (Tuwunel), and HTTP File System (MinIO).

## Project Structure

```
hiclaw/
├── manager/          # Manager Agent container (all-in-one: Higress + Tuwunel + MinIO + Element Web + OpenClaw)
├── worker/           # Worker Agent container (lightweight: OpenClaw + mc + mcporter)
├── install/          # One-click installation scripts
├── scripts/          # Utility scripts (replay-task.sh for sending tasks to Manager via Matrix)
├── hack/             # Maintenance scripts (mirror-images.sh for syncing base images to registry)
├── tests/            # Automated integration test suite (10 test cases)
├── .github/workflows/# CI/CD: build images, run tests, release
├── docs/             # User-facing documentation
├── design/           # Internal design documents and API specs
└── logs/             # Replay conversation logs (gitignored)
```

## Key Entry Points

### To understand the architecture
- Read [docs/architecture.md](docs/architecture.md) for system overview and component diagram
- Read [design/design.md](design/design.md) for full product design (Chinese)
- Read [design/poc-design.md](design/poc-design.md) for detailed implementation specs

### To build and run
- [Makefile](Makefile) -- unified build/test/push/install/replay interface (`make help` for all targets)
- [docs/quickstart.md](docs/quickstart.md) -- end-to-end guide from zero to working team
- [install/hiclaw-install.sh](install/hiclaw-install.sh) -- the installation script
- [scripts/replay-task.sh](scripts/replay-task.sh) -- send tasks to Manager via Matrix CLI

### Local full build (from source)

The image dependency chain is: `openclaw-base` → `manager` / `worker`. CoPaw worker is independent.

By default, `OPENCLAW_BASE_IMAGE` points to the remote registry (`higress-registry.cn-hangzhou.cr.aliyuncs.com/higress/openclaw-base`). When building locally from a modified `openclaw-base`, you **must** override it to the local image name so that manager/worker actually use your local base:

```bash
# Step 1: Build openclaw-base
make build-openclaw-base

# Step 2: Build manager, worker, copaw-worker using the LOCAL base
make build-manager build-worker build-copaw-worker \
    OPENCLAW_BASE_IMAGE=hiclaw/openclaw-base \
    OPENCLAW_BASE_VERSION=latest
```

**Common pitfall**: Running `make build-manager build-worker OPENCLAW_BASE_VERSION=latest` without `OPENCLAW_BASE_IMAGE=hiclaw/openclaw-base` will pull the remote registry's `:latest` tag instead of using the locally-built image. Always set both variables together for local builds.

**Proxy support**: If behind an HTTP proxy, pass proxy build args. This covers APT, PIP, NPM and all other network access — no mirror args needed:
```bash
PROXY_ARGS="--build-arg HTTP_PROXY=http://host.containers.internal:1087 \
    --build-arg HTTPS_PROXY=http://host.containers.internal:1087 \
    --build-arg http_proxy=http://host.containers.internal:1087 \
    --build-arg https_proxy=http://host.containers.internal:1087"

make build-embedded build-manager build-worker build-copaw-worker DOCKER_BUILD_ARGS="${PROXY_ARGS}"
```
Note: use `host.containers.internal` for Podman on macOS, `host.docker.internal` for Docker Desktop.

**China build acceleration (without proxy)**: All Dockerfiles default to official sources. For builds in China without proxy, pass mirror args:
```bash
# APT mirror (for Ubuntu/Debian-based images: openclaw-base, copaw, manager-copaw, embedded)
make build-embedded DOCKER_BUILD_ARGS="--build-arg APT_MIRROR=mirrors.aliyun.com"

# PIP mirror (for Python-based images: copaw, manager-copaw)
make build-copaw-worker DOCKER_BUILD_ARGS="--build-arg APT_MIRROR=mirrors.aliyun.com --build-arg PIP_INDEX_URL=https://mirrors.aliyun.com/pypi/simple/"

# NPM mirror (for Node.js-based images: openclaw-base)
make build-openclaw-base DOCKER_BUILD_ARGS="--build-arg APT_MIRROR=mirrors.aliyun.com --build-arg NPM_REGISTRY=https://registry.npmmirror.com/"
```

### To modify the Manager container
- [manager/Dockerfile](manager/Dockerfile) -- multi-stage build definition
- [manager/supervisord.conf](manager/supervisord.conf) -- process orchestration
- [manager/scripts/init/](manager/scripts/init/) -- container startup scripts (supervisord)
- [manager/scripts/lib/](manager/scripts/lib/) -- shared libraries (base.sh, container-api.sh)
- [manager/configs/](manager/configs/) -- init-time configuration templates

### To modify the Worker container (OpenClaw)
- [worker/Dockerfile](worker/Dockerfile) -- build definition (Node.js 22 from build stage)
- [worker/scripts/worker-entrypoint.sh](worker/scripts/worker-entrypoint.sh) -- startup logic

### To modify the Worker container (CoPaw)
- [copaw/Dockerfile](copaw/Dockerfile) -- build definition (Python 3.11)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentscope-ai/HiClaw](https://github.com/agentscope-ai/HiClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
