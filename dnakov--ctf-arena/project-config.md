---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CTF Arena is a competitive code golf platform where users compete for the lowest instruction count per challenge. It features:

1. **Sandbox** (`sandbox/`) - QEMU-based execution with deterministic instruction counting and syscall tracking
2. **API** (`api/`) - Rust HTTP service with GitHub OAuth, challenges, and leaderboards
3. **Compile Worker** (`compile-worker/`) - Compiles source code to static binaries
4. **Compiler** (`compiler/`) - Docker image with 26+ language compilers
5. **Worker** (`worker/`) - Executes binaries in sandboxed QEMU environment
6. **Web Frontend** (`web/`) - Svelte 5 + SvelteKit UI with Monaco editor
7. **Kubernetes** (`k8s/`) - Deployment manifests for local (kind) and production

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│  Svelte 5 Frontend (web/)                                               │
│  ┌─────────────────┐  ┌─────────────┐  ┌───────────────────────────────┐│
│  │ Monaco Editor   │  │ Challenge   │  │ Results Panel                 ││
│  │ (code input)    │  │ Selector    │  │ - Instructions: 2,008         ││
│  │                 │  │             │  │ - Syscalls: 5                 ││
│  │                 │  │             │  │ - Memory: 256 KB              ││
│  └─────────────────┘  └─────────────┘  └───────────────────────────────┘│
└───────────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────────────┐
│ API (Rust)      │────▶│ NATS JetStream  │────▶│ Compile Workers         │
│ /compile        │     │ COMPILES stream │     │ + Docker compiler image │
│ /submit         │◀────│ binaries KV     │◀────│                         │
│ /challenges     │     │                 │     └─────────────────────────┘
│ /auth/github    │     └─────────────────┘
└─────────────────┘            │
        │                      ▼
        │               ┌─────────────────────────┐
        │               │ Execute Workers         │
        │               │ + QEMU Sandbox          │
        ▼               └─────────────────────────┘
┌─────────────────┐
│ PostgreSQL      │
│ - users         │
│ - sessions      │
│ - challenges    │
│ - leaderboard   │
└─────────────────┘
```

## Quick Start

### Prerequisites
- Docker with buildx support
- kubectl + kind (for local k8s)
- Rust toolchain (for local API development)
- Node.js 20+ (for frontend development)

### Local Development with Kubernetes

```bash
# 1. Create kind cluster with local registry (first time only)
./scripts/kind-with-registry.sh

# 2. Build and push all images
docker build -t localhost:5001/ctf-api:latest ./api && docker push localhost:5001/ctf-api:latest
docker build -t localhost:5001/ctf-worker:latest ./worker && docker push localhost:5001/ctf-worker:latest
docker build -t localhost:5001/ctf-web:latest ./web && docker push localhost:5001/ctf-web:latest
docker build -t localhost:5001/compile-worker:latest ./compile-worker && docker push localhost:5001/compile-worker:latest
docker build --platform linux/amd64 -t localhost:5001/sandbox:latest ./sandbox && docker push localhost:5001/sandbox:latest

# 3. Build compiler image (large, ~15-20GB)
docker build --platform linux/amd64 -t compiler:latest ./compiler

# 4. Apply k8s manifests
kubectl apply -k k8s/overlays/local

# 5. Wait for pods to be ready
kubectl wait --for=condition=ready pod -l app=ctf-api -n ctf-arena --timeout=120s
kubectl wait --for=condition=ready pod -l app=ctf-worker -n ctf-arena --timeout=120s

# 6. Load images into worker DinD sidecars
WORKER_POD=$(kubectl get pods -n ctf-arena -l app=ctf-worker -o jsonpath='{.items[0].metadata.name}')
docker save localhost:5001/sandbox:latest | kubectl exec -n ctf-arena -i "$WORKER_POD" -c dind -- docker load

COMPILE_POD=$(kubectl get pods -n ctf-arena -l app=compile-worker -o jsonpath='{.items[0].metadata.name}')
docker save compiler:latest | kubectl exec -n ctf-arena -i "$COMPILE_POD" -c dind -- docker load

# 7. Start port forwards
kubectl port-forward -n ctf-arena svc/ctf-api 3000:3000 &
kubectl port-forward -n ctf-arena svc/ctf-web 8080:80 &

# 8. Verify
curl http://localhost:3000/health
```

**Access:**
- Web UI: http://localhost:8080
- API: http://localhost:3000

## Deploy Changes

After making code changes:

```bash
# Build and push changed images
docker build -t localhost:5001/ctf-api:latest ./api && docker push localhost:5001/ctf-api:latest
docker build -t localhost:5001/ctf-worker:latest ./worker && docker push localhost:5001/ctf-worker:latest
docker build -t localhost:5001/ctf-web:latest ./web && docker push localhost:5001/ctf-web:latest

# Restart deployments
kubectl rollout restart deployment/ctf-api deployment/ctf-worker deployment/ctf-web -n ctf-arena

# Wait for rollout
kubectl rollout status deployment/ctf-api deployment/ctf-worker deployment/ctf-web -n ctf-arena --timeout=120s

# Reload sandbox image into worker DinD

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dnakov/ctf-arena](https://github.com/dnakov/ctf-arena) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
