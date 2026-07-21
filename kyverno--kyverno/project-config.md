---
trigger: always_on
description: This file provides context for AI coding agents working on the Kyverno codebase.
---

# AGENTS.md — Kyverno

This file provides context for AI coding agents working on the Kyverno codebase.

## Project Overview

Kyverno is a Kubernetes-native policy engine for security, compliance, automation, and governance through policy-as-code. It validates, mutates, generates, and cleans up Kubernetes resources using admission controls and background scans, and verifies container image signatures for supply chain security.

- **Language:** Go
- **Module:** `github.com/kyverno/kyverno`
- **License:** Apache 2.0
- **Go version:** See `go.mod` for the current version

## Repository Structure

```
api/                  # Kubernetes API type definitions (CRDs)
  kyverno/            #   kyverno.io API group (v1, v1beta1, v2, v2alpha1, v2beta1)
  policyreport/       #   wgpolicyk8s.io API group
  reports/            #   reports.kyverno.io API group
cmd/                  # Entry points for all binaries
  kyverno/            #   Main admission controller
  kyverno-init/       #   Init container (kyvernopre) — pre-flight resource cleanup
  cli/                #   Kyverno CLI (kubectl-kyverno)
  cleanup-controller/ #   Cleanup controller
  reports-controller/ #   Reports controller
  background-controller/ # Background controller (generate/mutate existing)
  readiness-checker/  #   Readiness checker
  internal/           #   Shared internal cmd helpers
  tools/              #   Internal tooling (webhook-cleanup, etc.)
pkg/                  # Core library code
  engine/             #   Policy engine (rule evaluation, matching, context)
  webhooks/           #   Admission webhook handlers
  controllers/        #   Controller implementations
  cel/                #   CEL-based policy evaluation
  client/             #   Generated Kubernetes clientset, listers, informers
  clients/            #   Client wrappers with tracing/metrics
  config/             #   Runtime configuration
  toggle/             #   Feature flags
  logging/            #   Structured logging utilities
  metrics/            #   Prometheus metrics
  validation/         #   Policy validation logic
  autogen/            #   Auto-generation of rules for Pod controllers
  cosign/             #   Cosign image signature verification
  notary/             #   Notary image signature verification
  utils/              #   Shared utilities
ext/                  # Small standalone utility packages
charts/               # Helm charts
  kyverno/            #   Main Kyverno chart
  kyverno-policies/   #   Default policies chart
config/               # CRD manifests and install manifests
  crds/               #   Generated CRD YAML files
test/                 # Tests
  cli/                #   CLI test cases (kyverno test)
  conformance/        #   Conformance / e2e tests (chainsaw)
  fuzz/               #   Fuzz tests
  policy/             #   Policy test fixtures
docs/                 # Internal developer documentation
  dev/                #   API design, controllers, logging, feature flags, reports
scripts/              # Build and CI scripts
hack/                 # Code generation helpers
```

## Build System

The project uses `make` extensively. Tools are auto-installed into `.tools/` on first use.

### Key Build Commands

| Command | Description |
|---|---|
| `make build-all` | Build all binaries |
| `make build-kyverno` | Build the main kyverno binary → `cmd/kyverno/kyverno` |
| `make build-kyverno-init` | Build kyvernopre binary → `cmd/kyverno-init/kyvernopre` |
| `make build-cli` | Build CLI binary → `cmd/cli/kubectl-kyverno/kubectl-kyverno` |
| `make build-cleanup-controller` | Build cleanup controller binary |
| `make build-reports-controller` | Build reports controller binary |
| `make build-background-controller` | Build background controller binary |
| `make install-tools` | Install all development tools into `.tools/` |
| `make clean-tools` | Remove installed tools |

### Formatting & Linting

| Command | Description |
|---|---|
| `make fmt` | Run `go fmt ./...` |
| `make vet` | Run `go vet ./...` |
| `make imports` | Fix imports with `goimports` |
| `make fmt-check` | Verify formatting (fails if diff is non-empty) |
| `make imports-check` | Verify imports (fails if diff is non-empty) |
| `make unused-package-check` | Run `go mod tidy` check |

Linting is configured via `.golangci.yml` (golangci-lint v2). Enabled linters include `gosec`, `misspell`, `paralleltest`, `unconvert`, `errname`, `importas`, and others. The `importas` linter enforces specific import alias conventions — see `.golangci.yml` for the full alias rules.

Formatters enabled: `gci`, `gofmt`, `gofumpt`, `goimports`.

**Pre-commit checklist (required for code changes):**

- Run `make imports fmt` before committing.
- Run `make imports-check fmt-check` and ensure both pass.

**Pre-PR checks (required before opening/updating a PR):**

- Run `make codegen-all-code` then `make verify-codegen`.
- Run `./.tools/golangci-lint run` (install first with `make install-tools` if needed).


### Testing

| Command | Description |
|---|---|
| `make test-unit` | Run all unit tests with race detector and coverage |
| `make test-cli` | Run all CLI tests |
| `make test-cli-local` | Run local CLI test suite |
| `make test-clean` | Clear Go test cache |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kyverno/kyverno](https://github.com/kyverno/kyverno) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
