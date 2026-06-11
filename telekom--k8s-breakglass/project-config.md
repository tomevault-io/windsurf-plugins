---
trigger: always_on
description: Go-based Kubernetes controller (controller-runtime) + Vue/TypeScript frontend (Vite). Hub-and-spoke topology where a central breakglass service manages temporary privilege escalations across multiple Kubernetes clusters.
---

# Breakglass Controller - AI Coding Instructions

## Architecture Overview

Go-based Kubernetes controller (controller-runtime) + Vue/TypeScript frontend (Vite). Hub-and-spoke topology where a central breakglass service manages temporary privilege escalations across multiple Kubernetes clusters.

**Core Components:**
- `cmd/main.go` — Entry point with 5 deployment patterns (monolith, webhook-only, api-only, frontend-only, cleanup-only)
- `api/v1alpha1/` — CRD types with kubebuilder markers + admission webhooks
- `pkg/breakglass/` — Session/escalation business logic, approval workflows, cleanup routines
- `pkg/api/` — Gin HTTP server, REST endpoints (shared with frontend)
- `pkg/webhook/` — Kubernetes authorization webhook (SubjectAccessReview)
- `pkg/reconciler/` — Controller-runtime manager, metrics, health probes, indexers
- `frontend/` — Vue 3 + Vite web application

**CRD Resources:** `BreakglassEscalation`, `BreakglassSession`, `ClusterConfig`, `IdentityProvider`, `MailProvider`, `DenyPolicy`, `DebugSession`, `DebugSessionTemplate`, `DebugPodTemplate`

## Essential Developer Workflows

```bash
# After modifying api/v1alpha1/*.go types - ALWAYS run both:
make generate    # DeepCopy methods (zz_generated.deepcopy.go)
make manifests   # CRDs, webhooks, RBAC in config/crd/bases/

# Testing & linting
make test        # Unit tests (excludes e2e)
make lint        # golangci-lint (auto-installs to ./bin)

# Local development (kind cluster)
make docker-build-dev
kind create cluster
kind load docker-image breakglass:dev
make install       # Install CRDs
make deploy_dev    # Deploy with Keycloak + MailHog

# Full E2E environment (see "Running E2E Tests Locally" section for details)
make docker-build IMG=breakglass:e2e UI_FLAVOUR=telekom
SKIP_BUILD=true SKIP_PROXY=true IMAGE=breakglass:e2e UI_FLAVOUR=telekom ./e2e/kind-setup-single.sh

# Frontend development
cd frontend
npm run dev:mock   # Mock API, no backend needed
npm run dev        # Against real backend (port 5173 → 8080)
npm test           # Vitest
```

## Critical Conventions

1. **CRD Changes**: Edit `api/v1alpha1/*_types.go`, run `make generate && make manifests`, commit generated files (`zz_generated.deepcopy.go`, `config/crd/bases/*.yaml`).

2. **Webhooks**: Use `//+kubebuilder:webhook` markers. Cert generation at startup (`pkg/cert/`). Check `config/webhook/` for kustomize overlays.

3. **Required CRs**: `IdentityProvider` and `MailProvider` CRDs must exist for normal operation. Tests/local runs expect these.

4. **Component Flags**: Environment variables or CLI flags control components:
   - `ENABLE_FRONTEND`, `ENABLE_API`, `ENABLE_WEBHOOKS`, `ENABLE_CLEANUP`
   - `WEBHOOKS_METRICS_BIND_ADDRESS` for separate webhook metrics

5. **Client Usage**: Prefer `reconcilerMgr.GetClient()` (cached) for reads; uncached clients for webhooks/startup (see `cmd/main.go`).

6. **Tooling**: Local tools in `./bin/` via Makefile. Use `make kustomize`, `make controller-gen` for pinned versions.

7. **Linting (MANDATORY)**: Before submitting any code changes:
   - Run `make lint` and fix ALL errors before committing
   - Use `http.MethodGet`, `http.MethodPost`, etc. instead of string literals like `"GET"`, `"POST"`
   - Remove unnecessary type conversions (e.g., `string(x) == string(y)` when types are compatible)
   - CI will reject PRs with lint failures
   - For frontend changes: Run `cd frontend && npm run lint` and fix errors
    - For frontend changes: Run `cd frontend && npm run typecheck` (or `npm run build`) to catch TS/template errors
    - For frontend changes: Run `cd frontend && npm test` and fix failing unit tests before opening PRs

8. **Documentation (MANDATORY)**: Documentation MUST be updated with every code change:
   - API changes → Update `docs/api-reference.md` with endpoint signatures, request/response formats
   - CRD changes → Update relevant docs in `docs/` (e.g., `breakglass-session.md`, `cluster-config.md`)
   - New features → Add to `docs/advanced-features.md` or create new doc, update `docs/README.md` index
   - Configuration changes → Update `docs/configuration-reference.md` and `docs/cli-flags-reference.md`
   - Helm chart changes → Update `charts/escalation-config/README.md` and inline `values.yaml` comments
   - **CHANGELOG updates** → Update `CHANGELOG.md` for every user-facing change (see below)

9. **CHANGELOG Updates (MANDATORY)**: Every PR with user-facing changes MUST update `CHANGELOG.md`:
   - Add entries under `## [Unreleased]` section in the appropriate category
   - Categories: `Added`, `Changed`, `Deprecated`, `Removed`, `Fixed`, `Security`
   - Format: `- Brief description of change (PR #123)` - include PR number when available
   - New features → `Added`
   - Breaking changes → `Changed` with migration notes
   - Bug fixes → `Fixed`
   - Security patches → `Security`
   - Dependency updates → Generally skip unless security-related or breaking
   - When releasing: Move `Unreleased` content to new version section with date
   - Follow [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) format


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [telekom/k8s-breakglass](https://github.com/telekom/k8s-breakglass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
