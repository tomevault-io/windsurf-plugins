---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

KubeBolt is an instant Kubernetes monitoring platform — full cluster visibility in under 2 minutes with zero configuration. Go backend + React frontend monorepo. Supports multi-cluster switching and Gateway API resources.

## Build & Run Commands

### Backend (Go)
```bash
cd apps/api && go run cmd/server/main.go --kubeconfig ~/.kube/config  # Run dev server (port 8080)
cd apps/api && go build ./...                                          # Build
cd apps/api && go test ./...                                           # Run tests
cd apps/api && go test ./internal/insights/...                         # Run single package tests
```

### Local stack with empty kubeconfig
```bash
make dev-clean       # API + Web on host with /tmp/kb-empty-kubeconfig.yaml (no contexts)
make dev-api-clean   # API only with empty kubeconfig
```
Use these when testing the persistent-registry boot-restore path or
the no-clusters / waiting-for-agent empty-state UX without touching
your real `~/.kube/config`. The empty kubeconfig is regenerated on
every invocation so accidental edits don't persist.

### Frontend (React)
```bash
cd apps/web && npm install    # Install dependencies
cd apps/web && npm run dev    # Vite dev server (port 5173)
cd apps/web && npm run build  # Production build (TypeScript check + Vite)
```

### Docker Compose (full stack)
```bash
# Remote clusters (EKS, GKE, AKS) — works directly:
kubectl config use-context my-cluster
cd deploy && docker compose up -d

# Docker Desktop K8s — needs kubeconfig rewrite (127.0.0.1 → kubernetes.docker.internal):
kubectl config use-context docker-desktop
./deploy/docker-kubeconfig.sh   # generates /tmp/docker-kubeconfig
cd deploy && docker compose up -d

# Rebuild after code changes:
docker compose -f deploy/docker-compose.yml up -d --build
```
Frontend on http://localhost:3000 (nginx proxies /api and /ws to backend).
EKS clusters require `~/.aws` mounted (already in compose) with an active AWS session.

### Helm Chart
```bash
helm install kubebolt oci://ghcr.io/clm-cloud-solutions/kubebolt/helm/kubebolt
kubectl port-forward svc/kubebolt 3000:80
```
When deployed via Helm, the API auto-detects in-cluster config (ServiceAccount token). The web container uses `API_BACKEND` env var (set by Helm template) to point nginx at the correct API service name.

## Architecture

### Go Workspace Monorepo

Uses `go.work` with three modules:
- `apps/api` — main backend server

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [clm-cloud-solutions/kubebolt](https://github.com/clm-cloud-solutions/kubebolt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
