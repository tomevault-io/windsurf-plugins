---
trigger: always_on
description: Kubernetes-native AI automation platform that orchestrates agentic sessions through containerized microservices. Built with Go (API server, control plane), NextJS + Shadcn (UI), and Python (runner). PostgreSQL is the source of truth; the control plane reconciles via gRPC watch streams.
---

# Agent Control Plane

Kubernetes-native AI automation platform that orchestrates agentic sessions through containerized microservices. Built with Go (API server, control plane), NextJS + Shadcn (UI), and Python (runner). PostgreSQL is the source of truth; the control plane reconciles via gRPC watch streams.

> Some RBAC manifests still reference the `vteam.ambient-code` API group for backward compatibility.

## Structure

- `components/ambient-api-server/` - Go REST API microservice (rh-trex-ai framework), PostgreSQL-backed
- `components/ambient-control-plane/` - Go service, watches API server via gRPC and reconciles sessions into K8s Jobs
- `components/ambient-ui/` - NextJS + Shadcn web UI for session management and monitoring
- `components/ambient-mcp/` - MCP server integration
- `components/runners/ambient-runner/` - Python runner executing Claude Code CLI in Job pods
- `components/ambient-cli/` - Go CLI (`acpctl`), manages agentic sessions from the command line
- `components/ambient-sdk/` - Go, Python, and TypeScript client SDKs generated from the OpenAPI spec
- `components/credential-sidecars/` - Per-provider credential sidecar containers (GitHub, Jira, K8s, Google)
- `components/manifests/` - Kustomize-based deployment manifests and overlays
- `docs/` - Astro Starlight documentation site
- `specs/` - Desired state of the system ([platform](specs/platform/), [security](specs/security/), [ui](specs/ui/), [standards](specs/standards/))
- `skills/` - Agent skills: [reconcile](skills/build/reconcile), [spec](skills/plan/spec), [full-stack-pipeline](skills/build/full-stack-pipeline), [dev-cluster](skills/build/dev-cluster), [pr-test](skills/test/pr-test), [deploy-cluster](skills/deploy/deploy-cluster), [review](skills/review/acp-review-guidance), [tooling](skills/tooling/)
- `apm.yml` - APM manifest declaring upstream skill dependencies (fleet-sdlc)
- `.claude/skills/` - APM-installed upstream skills (gitignored, run `apm install`)
- `.claude/commands/` - APM-installed upstream commands (gitignored)

## Key Files

- Control plane reconciler: `components/ambient-control-plane/internal/reconciler/kube_reconciler.go`
- K8s client init: `components/ambient-control-plane/internal/config/config.go`
- Runner entry point: `components/runners/ambient-runner/main.py`
- RBAC: ClusterRoles in `components/manifests/base/rbac/` grant `vteam.ambient-code` API group access (legacy CRs created as side-effects, not used as source of truth)

## Session Flow

```
User Creates Session → API Server Persists to DB → Control Plane Spawns Job →
Pod Runs AI Agent → Results Stream to API Server → UI Displays Progress
```

## SDLC Workflow

The development lifecycle follows 6 steps, each backed by a skill:

```
0. /reconcile             — autonomous spec-to-code reconciliation (build/reconcile)
1. /spec                  — define desired state (plan/spec)
2. /full-stack-pipeline   — build the feature (build/full-stack-pipeline)
3. /dev-cluster           — test locally in Kind (build/dev-cluster)
4. /pr-test               — deploy PR to OpenShift (test/pr-test)
5. /deploy-cluster        — ship to production (deploy/deploy-cluster)
```

`/reconcile` is the top-level entrypoint. It reads `skills/RECONCILE.md` for checkpoint
state (coverage summary, gap table, wave plan), then executes waves to close gaps.
Idempotent: safe to run repeatedly. See `skills/RECONCILE.md` for current spec coverage
and the full gap table. Use individual skills for targeted work.

Support skills available at any point:
- `/acp-review-guidance` — PR review checklist
- `/pr-fixer` — auto-fix PR from review comments
- `/align` — convention health check
- `/memory` — project memory management

## Commands

```shell
make build-all                # Build all container images
make deploy                   # Deploy to cluster
make test-all                 # Run all tests
make lint                     # Lint code
make kind-up                  # Start local Kind cluster
make kind-rebuild              # Rebuild images + redeploy to running cluster
make kind-login                # Set kubectl context + configure acpctl
make dev-bootstrap             # Bootstrap developer workspace
make benchmark                # Run component benchmark harness
```

### Per-Component

```shell
# Control Plane (Go)
cd components/ambient-control-plane && gofmt -l . && go vet ./... && golangci-lint run

# API Server (Go)
cd components/ambient-api-server && gofmt -l . && go vet ./... && golangci-lint run

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
make benchmark COMPONENT=ambient-control-plane MODE=cold
```

Benchmark notes:

- `FORMAT=tsv` is preferred for agents to minimize token usage
- `warm` measures rebuild proxies, not browser-observed hot reload latency

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openshift-online/agent-control-plane](https://github.com/openshift-online/agent-control-plane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
