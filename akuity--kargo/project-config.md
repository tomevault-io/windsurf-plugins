---
trigger: always_on
description: Kargo is a Kubernetes-native continuous promotion platform for GitOps workflows.
---

# Kargo

Kargo is a Kubernetes-native continuous promotion platform for GitOps workflows.
Warehouses watch for new artifacts (container images, Git commits, Helm charts),
bundle them together as Freight, and promote that as a unit through a pipeline
of Stages.

Agent should reference `docs/docs/` if a more comprehensive understanding of the
domain is needed.

## Project Layout

| Path | Description |
|------|-------------|
| `api/v1alpha1/` | CRD types (Warehouse, Freight, Stage, Promotion, Project, etc.) and protobuf specs. Separate Go module |
| `cmd/controlplane/` | Back end entry point -- single binary with subcommands: `api`, `controller`, `management-controller`, `kubernetes-webhooks`, `external-webhooks`, `garbage-collector` |
| `cmd/cli/` | CLI entry point |
| `pkg/server/` | API server handlers. Two coexisting APIs: **ConnectRPC** (DEPRECATED, removal in v1.12.0; still used by UI -- avoid investing in fixes or enhancements) and **REST API** (the replacement; used by CLI; UI has not yet migrated) |
| `pkg/cli/` | CLI -- Cobra-based, subcommands in `pkg/cli/cmd/`, REST API client in `pkg/cli/client/` |
| `pkg/controller/` | Kubernetes controllers for Kargo resources |
| `pkg/promotion/` | Promotion engine and step runner |
| `pkg/promotion/runner/builtin/` | Built-in promotion steps (git, helm, kustomize, etc.) |
| `pkg/gitprovider/` | Git provider integrations (GitHub, GitLab, Gitea, BitBucket) |
| `pkg/image/` | Container image registry operations |
| `pkg/credentials/` | Secrets/credentials management |
| `pkg/webhook/` | Webhook handlers |
| `ui/` | React/TypeScript frontend (Vite + Ant Design + TanStack Query) |
| `charts/kargo/` | Helm chart -- primary installation method |

## Build & Development

### Prerequisites

Make, Go, Node.js, pnpm, and Docker are the primary prerequisites. Most other
tools fall into two buckets:

- **Go-based tools** (buf, controller-gen, go-to-protobuf, protoc-gen-gogo,
  protoc-gen-doc, goimports, swag, go-swagger, oapi-codegen, ctlptl, kind) are
  declared as `tool` directives in the root `go.mod` and invoked via
  `go tool <name>`. No install step is needed -- `go tool` builds them on demand
  from the module.
- **Prebuilt-binary tools** (golangci-lint, helm, protoc, quill, tilt, k3d, jq)
  are installed automatically into `hack/bin/` by Make targets (see
  `hack/tools.mk`). golangci-lint stays a prebuilt binary rather than a
  `go tool` because building it from source is slow and can drift from the
  release binary's lint results; its version is pinned in `hack/tools.mk`.

### Common commands

```bash
make lint-go              # Lint Go code (golangci-lint)
make lint                 # Lint everything (Go, proto, charts, UI)
make format-go            # Auto-format Go code
make test-unit            # Run unit tests (with -race)
make build-cli            # Build CLI binary
make codegen              # Run all code generation
make hack-build-dev-tools # Build dev container with all tools
```

Containerized equivalents (no local tool installs needed):

```bash
make hack-lint-go
make hack-test-unit
make hack-codegen
```

Build the Kargo image:

```bash
make hack-build           # Build container image (kargo:dev)
```

There is seldom a need to do so directly.

### Local development with Tilt

```bash
make hack-kind-up          # Create local K8s cluster (or hack-k3d-up)
make hack-tilt-up          # Start local dev environment
```

`hack-kind-up` / `hack-k3d-up` are not needed if using OrbStack or Docker
Desktop's built-in Kubernetes clusters.

Tools like `tilt`, `ctlptl`, `kind`, `k3d`, and `helm` are auto-installed into
`hack/bin/` by these targets -- no manual installation needed. Tilt also handles
installing prerequisites (cert-manager, Argo CD, Argo Rollouts) idempotently.

- Tilt compiles back end on source changes
- **Manual trigger mode** used for re-deploying to the cluster -- trigger
  re-deployment from the Tilt UI (http://localhost:10350) or `hack/bin/tilt
  trigger <component>`
- API: localhost:30081, UI: localhost:30082, External webhooks: localhost:30083
- Argo CD: localhost:30080 (admin/admin)
- Kargo admin password: `admin`
- `make hack-tilt-down` to undeploy Kargo (preserves prerequisites)
- `make hack-kind-down` / `make hack-k3d-down` to destroy the cluster entirely

### Code generation

Run `make codegen` (or `make hack-codegen`) after modifying:

- API types in `api/v1alpha1/`
- Protobuf definitions
- Swagger annotations in `pkg/server/`
- JSON schemas for promotion step configs

This generates: protobuf bindings, CRDs, deepcopy methods, OpenAPI specs,
TypeScript API clients, and Go client code.

## Code Conventions

### Principles

- Clear over clever
- Simple over complex -- don't over-engineer or prematurely optimize
- Break large problems into small, well-defined pieces
- Structure code for testability
- Always include unit tests for new and modified code
- Never disable or skip a failing test -- fix the underlying problem

### Go style

- **Line length**: soft limit 80, hard limit 120. Don't sacrifice readability
  to hit 80 -- a few characters over is fine. Use `nolint: lll` when exceeding
  120 is truly unavoidable
- **Errors**: stdlib `errors` package only; never `github.com/pkg/errors`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [akuity/kargo](https://github.com/akuity/kargo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
