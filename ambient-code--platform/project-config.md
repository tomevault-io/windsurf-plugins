---
trigger: always_on
description: Kubernetes-native AI automation platform that orchestrates agentic sessions through containerized microservices. Built with Go (backend, operator), NextJS + Shadcn (frontend), Python (runner), and Kubernetes CRDs.
---

# Ambient Code Platform

Kubernetes-native AI automation platform that orchestrates agentic sessions through containerized microservices. Built with Go (backend, operator), NextJS + Shadcn (frontend), Python (runner), and Kubernetes CRDs.

> Technical artifacts still use "vteam" for backward compatibility.

## Structure

- `components/backend/` - Go REST API (Gin), manages K8s Custom Resources with multi-tenant project isolation
- `components/frontend/` - NextJS web UI for session management and monitoring
- `components/operator/` - Go Kubernetes controller, watches CRDs and creates Jobs
- `components/runners/ambient-runner/` - Python runner executing Claude Code CLI in Job pods
- `components/ambient-cli/` - Go CLI (`acpctl`), manages agentic sessions from the command line
- `components/public-api/` - Stateless HTTP gateway, proxies to backend (no direct K8s access)
- `components/ambient-api-server/` - Go REST API microservice (rh-trex-ai framework), PostgreSQL-backed
- `components/ambient-sdk/` - Go + Python client SDK for the platform's public REST API
- `components/open-webui-llm/` - Open WebUI LLM integration
- `components/manifests/` - Kustomize-based deployment manifests and overlays
- `e2e/` - Cypress end-to-end tests
- `docs/` - Astro Starlight documentation site
- `specs/` - Desired state of the system ([sessions](specs/sessions/), [agents](specs/agents/), [control-plane](specs/control-plane/), [integrations](specs/integrations/), [standards](specs/standards/))
- `workflows/` - Agent-consumable procedures ([sessions](workflows/sessions/), [control-plane](workflows/control-plane/), [integrations](workflows/integrations/))
- `skills/` - [Agent Skills](https://agentskills.io) (`.claude/skills` symlinks here; domain symlinks in `specs/{domain}/.agents/skills`)

## Key Files

- CRD definitions: `components/manifests/base/crds/agenticsessions-crd.yaml`, `projectsettings-crd.yaml`
- Session lifecycle: `components/backend/handlers/sessions.go`, `components/operator/internal/handlers/sessions.go`
- Auth & RBAC middleware: `components/backend/handlers/middleware.go`
- K8s client init: `components/operator/internal/config/config.go`
- Runner entry point: `components/runners/ambient-runner/main.py`
- Route registration: `components/backend/routes.go`
- Frontend API layer: `components/frontend/src/services/api/`, `src/services/queries/`

## Session Flow

```
User Creates Session → Backend Creates CR → Operator Spawns Job →
Pod Runs Claude CLI → Results Stored in CR → UI Displays Progress
```

## Commands

```shell
make build-all                # Build all container images
make deploy                   # Deploy to cluster
make test                     # Run tests
make lint                     # Lint code
make kind-up                  # Start local Kind cluster
make kind-rebuild              # Rebuild images + redeploy to running cluster
make kind-login                # Set kubectl context + configure acpctl
make dev-bootstrap             # Bootstrap developer workspace
make test-e2e-local           # Run E2E tests against Kind
make benchmark                # Run component benchmark harness
```

### Per-Component

```shell
# Backend / Operator (Go)
cd components/backend && gofmt -l . && go vet ./... && golangci-lint run
cd components/operator && gofmt -l . && go vet ./... && golangci-lint run

# Frontend
cd components/frontend && npm run build  # Must pass with 0 errors, 0 warnings

# Runner (Python)
cd components/runners/ambient-runner && uv venv && uv pip install -e .

# Docs
cd docs && npm run dev  # http://localhost:4321
```

### Benchmarking

```shell
# Human-friendly summary
make benchmark

# Agent / automation friendly output
make benchmark FORMAT=tsv

# Single component
make benchmark COMPONENT=frontend MODE=cold
```

Benchmark notes:

- `frontend` requires **Node.js 20+**
- `FORMAT=tsv` is preferred for agents to minimize token usage
- `warm` measures rebuild proxies, not browser-observed hot reload latency
- See `scripts/benchmarks/README.md` for semantics and caveats

## Critical Conventions

Cross-cutting rules that apply across ALL components. Component-specific conventions live in
[specs/standards/](specs/standards/) (see [BOOKMARKS.md](BOOKMARKS.md) > Component Standards).

- **User token auth required**: All user-facing API ops use `GetK8sClientsForRequest(c)`, never the backend service account
- **No tokens in logs/errors/responses**: Use `len(token)` for logging, generic messages to users
- **OwnerReferences on all child resources**: Jobs, Secrets, PVCs must have controller owner refs
- **No `panic()` in production**: Return explicit `fmt.Errorf` with context
- **No `any` types in frontend**: Use proper types, `unknown`, or generic constraints
- **Feature flags strongly recommended**: Gate new features behind Unleash flags. Use `/unleash-flag` to set up
- **No new CRDs**: Existing CRDs (AgenticSession, ProjectSettings) are grandfathered. For new persistent storage, confirm with the user whether to use repo files or PostgreSQL — do not default to K8s custom resources
- **Conventional commits**: Squashed on merge to `main`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ambient-code/platform](https://github.com/ambient-code/platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
