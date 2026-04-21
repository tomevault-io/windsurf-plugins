---
trigger: always_on
description: This file helps AI coding agents (Copilot, Cursor, Cline, etc.) understand the Sympozium project structure and development workflow.
---

# AGENTS.md — Contributor Guide for AI Agents

This file helps AI coding agents (Copilot, Cursor, Cline, etc.) understand the Sympozium project structure and development workflow.

---

## Project Overview

Sympozium is a **Kubernetes-native agent orchestration platform** written in Go. Every AI agent runs as an ephemeral Kubernetes pod (Job), with policy enforcement via CRDs, admission webhooks, and RBAC. Communication flows through NATS JetStream and a filesystem-based IPC bridge.

- **Language:** Go 1.25+
- **Module:** `github.com/sympozium-ai/sympozium`
- **K8s API version:** `sympozium.ai/v1alpha1`

---

## Repository Layout

```
api/v1alpha1/           # CRD type definitions (SympoziumInstance, AgentRun, SympoziumPolicy, SkillPack, SympoziumSchedule, Ensemble)
cmd/
  agent-runner/         # Agent container — LLM loop + tool execution
  apiserver/            # HTTP + WebSocket API server
  controller/           # Controller manager (all reconcilers + routers)
  ipc-bridge/           # IPC bridge sidecar (fsnotify → NATS)
  web-proxy/            # Web proxy (OpenAI-compat API + MCP gateway)
  node-probe/           # Node probe DaemonSet (discovers inference providers on nodes)
  sympozium/             # CLI + TUI (Bubble Tea)
  webhook/              # Admission webhook server
channels/
  telegram/             # Channel pod — Telegram bot
  slack/                # Channel pod — Slack (Socket Mode + Events API)
  discord/              # Channel pod — Discord bot
  whatsapp/             # Channel pod — WhatsApp
config/
  crd/bases/            # Generated CRD YAML manifests
  manager/              # Controller manager deployment
  personas/             # Built-in Ensemble YAML definitions
  rbac/                 # RBAC roles
  samples/              # Sample CR YAML files
  skills/               # Built-in SkillPack YAML definitions
  policies/             # Built-in SympoziumPolicy presets
  webhook/              # Webhook configuration
images/                 # Dockerfiles for all components
internal/
  apiserver/            # API server implementation
  channel/              # Channel types
  controller/           # Reconcilers (AgentRun, SympoziumInstance, SympoziumPolicy, SympoziumSchedule, SkillPack, Ensemble) + routers (Channel, Schedule)
  eventbus/             # NATS JetStream client + topic constants
  ipc/                  # IPC bridge (fsnotify watcher, protocol, file handlers)
  orchestrator/         # Pod builder + spawner for agent Jobs
  session/              # Session store
  webhook/              # Policy enforcer
  webproxy/             # Web proxy handlers (OpenAI, MCP, rate limiting)
migrations/             # PostgreSQL schema migrations
test/integration/       # Integration test scripts (shell)
docs/                   # Design & contributor documentation
```

---

## Key CRDs

| CRD | Purpose |
|-----|---------|
| `SympoziumInstance` | An agent identity — provider config, model, enabled skills, channel bindings |
| `AgentRun` | A single agent invocation — task, result, phase lifecycle |
| `SympoziumPolicy` | Policy rules enforced by the admission webhook |
| `SkillPack` | Bundled skills (Markdown instructions) + optional sidecar container + RBAC |
| `SympoziumSchedule` | Cron-based recurring AgentRun creation (heartbeat, scheduled, sweep) |
| `Ensemble` | Pre-configured agent bundles — stamps out Instances, Schedules, and memory automatically |

Type definitions live in `api/v1alpha1/`. After modifying types, regenerate with:

```bash
make generate    # deepcopy + CRD manifests
make manifests   # CRD YAML only
```

---

## Development Environment Setup

### Prerequisites

- Go 1.25+
- Docker
- [Kind](https://kind.sigs.k8s.io/) (Kubernetes in Docker)
- kubectl
- An LLM API key (e.g. `OPENAI_API_KEY`)

### Create a Kind Cluster & Install Sympozium

```bash
# Create cluster
kind create cluster --name kind

# Install CRDs
make install

# Build all images
make docker-build TAG=v0.1.0

# Load images into Kind (all components)
for img in controller apiserver ipc-bridge webhook agent-runner web-proxy \
           channel-telegram channel-slack channel-discord channel-whatsapp \
           skill-k8s-ops skill-sre-observability skill-llmfit; do
  kind load docker-image ghcr.io/sympozium-ai/sympozium/$img:v0.1.0 --name kind
done

# Deploy the control plane
kubectl apply -k config/
```

### Build & Test Cycle

After code changes:

```bash
# Build everything
make build

# Run unit tests
make test

# Build specific image + reload into Kind
make docker-build-agent-runner TAG=v0.1.0
kind load docker-image ghcr.io/sympozium-ai/sympozium/agent-runner:v0.1.0 --name kind

# Restart the controller to pick up new images
kubectl rollout restart deployment sympozium-controller-manager -n sympozium-system
```

### Common Build Targets

```bash
make build              # Build all binaries
make test               # Run unit tests with race detector
make test-short         # Run short tests only
make test-integration   # Run all integration tests (requires Kind + API key)
make vet                # go vet
make fmt                # gofmt
make tidy               # go mod tidy
make docker-build       # Build all Docker images

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sympozium-ai/sympozium](https://github.com/sympozium-ai/sympozium) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
