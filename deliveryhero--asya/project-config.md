---
trigger: always_on
description: AI developer guidance for the Asya project.
---

# AGENTS.md

AI developer guidance for the Asya project.

## Project Overview

Asya is an Actor Mesh framework for running AI workloads on Kubernetes using
choreography (decentralized) instead of centralized orchestration. Actors
communicate by passing envelopes through message queues; routing is embedded in
each envelope, not managed by a central coordinator. Implemented with **Crossplane Compositions**
for declarative actor deployment with inline sidecar rendering.

Core components (all in `src/`):
- **asya-sidecar** (Go): envelope router injected into actor pods; Queue → Sidecar → Runtime → Sidecar → Next Queue
- **asya-runtime** (Python): lightweight socket server loaded via ConfigMap; executes user handler,
  returns result. Source of truth: `src/asya-runtime/asya_runtime.py` (single file, no deps).
  `deploy/helm-charts/asya-crossplane/files/asya_runtime.py` is a symlink — editing the source
  automatically reflects in the Crossplane chart's ConfigMap. No manual sync needed.
- **asya-gateway** (Go): optional MCP/HTTP gateway; exposes async actor pipelines as synchronous HTTP
- **asya-crew** (Python): system actors — `x-sink` (persist results), `x-sump` (DLQ handling),
  `x-pause` (checkpoint envelope to S3 and signal `paused`), `x-resume` (restore envelope from S3
  and re-inject into the mesh)
- **asya-lab** (Python): CLI tools (`asya mcp ...`, `asya flow ...`) for debugging and flow compilation
- **asya-testing** (Python): shared test fixtures and utilities
- **asya-state-proxy** (Go): optional sidecar that gives actors virtual persistent state via filesystem
  emulation; actors read/write `/state/...` paths, runtime intercepts Python file I/O and forwards to the
  proxy over Unix socket; proxy translates to actual storage backend (S3, GCS, Redis, NATS KV) with
  configurable LWW or CAS guarantees; actors remain stateless Deployments — no StatefulSets

**Project structure**:

```
asya/
├── src/
│   ├── asya-gateway/
│   ├── asya-sidecar/
│   ├── asya-runtime/
│   ├── asya-crew/
│   ├── asya-testing/
│   └── asya-cli/
├── deploy/helm-charts/
│   ├── asya-crew/          # pre-built generic actors (like x-sink, x-sump)
│   ├── asya-crossplane/    # contains AsyncActor XRD
│   └── asya-gateway/       # sync stateful HTTP gateway exposing actors and flows as MCP tools or A2A agents
├── testing/
│   ├── component/          # docker-compose
│   ├── integration/        # docker-compose
│   ├── e2e/                # local kind cluster
│   └── shared/             # shared docker-compose configurations
└── examples/
│   ├── asyas/              # sample XR definitions
    └── flows/              # sample Python flows
```

See [docs/reference/components/](docs/reference/components/) for component deep-dives.

**Examples** (`examples/`):
- `asyas/` — real-world AsyncActor CRD manifests; use as reference when writing or reviewing actor specs
- `flows/` — teaser flow DSL (Domain-Specific Language) files; comprehensive examples (including agentic flows) are in [examples/end-to-end/monorepo/](examples/end-to-end/monorepo/)
- `end-to-end/monorepo/` — full working examples organized by category (control-flow, agentic, resiliency, compiler-sugar, text-improver)

**Crossplane chart** (`deploy/helm-charts/asya-crossplane/`): Deploys XRDs, Compositions, and provider configurations for AsyncActor resource management. The `render-deployment` composition step renders the complete pod spec (runtime container + asya-sidecar + state proxies + volumes) using values from the chart's `sidecar:` block.

## Quick Reference

**Prerequisites**: uv (`curl -LsSf https://astral.sh/uv/install.sh | sh`), Go 1.24+, Python 3.13+, Docker, Make

```bash
make setup              # Install uv, pre-commit hooks, sync Go deps
make build              # Build all components
make build-images       # Build Docker images
make build-go           # Build Go components only
make test-unit          # Unit tests (Go + Python)
make test-component     # Component tests (Docker Compose)
make test-integration   # Integration tests (Docker Compose)
make test-e2e           # E2E tests (Kind cluster)
make lint               # Run all linters with auto-fix
make clean              # Remove build artifacts
```

Prefer `make <target>`. Add new Makefile targets instead of repeating raw commands.

## Testing Strategy
**Hierarchy**:
1. **Unit** (`make test-unit`): fast, no external deps — `src/{component}/tests/`
2. **Component** (`make test-component`): single component in Docker Compose — `testing/component/{component}/`
3. **Integration** (`make test-integration`): multi-component Docker Compose — `testing/integration/{suite}/`
4. **E2E** (`make test-e2e`): full stack in Kind cluster — `testing/e2e/`

**Trust CI**: Component, integration, and e2e tests cannot be parallelized across PRs. For multi-PR work,
push and observe CI logs rather than running all suites locally.

**Critical rules**:
- Unit tests must mock all external services
- Component/integration tests run inside Docker Compose — no port-forwarding
- Only E2E tests may use `kubectl port-forward`
- Prefer Docker Compose over Kind; use Kind only for K8s-specific features (CRDs, KEDA, Crossplane)

**E2E local debugging** (only when user explicitly permits Kind cluster access):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deliveryhero/asya](https://github.com/deliveryhero/asya) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
