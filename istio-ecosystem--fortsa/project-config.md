---
trigger: always_on
description: This file defines **non-negotiable engineering standards** for anyone (human or AI agent) making changes to this repository. It is optimized for **high Go code quality**, long-term maintainability, and safety when interacting with Kubernetes clusters.
---

# Agents Standards

## Purpose

This file defines **non-negotiable engineering standards** for anyone (human or AI agent) making changes to this repository. It is optimized for **high Go code quality**, long-term maintainability, and safety when interacting with Kubernetes clusters.

Fortsa is a Kubernetes operator. Small mistakes (reconcile loops, incorrect patching, unsafe retries, noisy logs) can have outsized impact in production. Treat this repo as **production-grade control-plane code**.

## Non-negotiables (read first)

- You **MUST** keep the operator **safe-by-default**:
  - **MUST NOT** cause uncontrolled reconcile storms.
  - **MUST NOT** introduce hot loops (tight polling, immediate requeues without backoff/jitter).
  - **MUST NOT** patch workloads repeatedly without cooldown/guards.
  - **MUST NOT** broaden RBAC needs without explicit justification and review.
- You **MUST** follow communication and documentation hygiene:
  - **NEVER** use emoji, or Unicode characters that emulate emoji (e.g. ✓, ✗).
  - **MUST** avoid redundant comments that are tautological/self-demonstrating (e.g. restating what a clearly named function does, or narrating code that is obvious at a glance).
- You **MUST** preserve the project’s architectural constraints:
  - **MUST NOT** introduce CRDs (this project explicitly aims for “no CRDs”).
  - **MUST** keep dependencies minimal and appropriate for controller-runtime/client-go ecosystems.
  - **MUST** keep the final artifact as a **single Go binary** suitable for a minimal container image.
- You **MUST** keep code formatted and lint-clean:
  - **MUST** run `make fmt`.
  - **MUST** run `make vet`.
  - **MUST** run `make lint` (golangci-lint).
  - **MUST** ensure tests relevant to your change pass (`make test`, and `make test-integration` / `make test-e2e*` when appropriate).

## Repository workflows (canonical commands)

Use the Makefile targets as the canonical workflow:

- **Formatting**: `make fmt`
- **Static analysis**: `make vet`
- **Lint**: `make lint` (and `make lint-fix` only for safe, mechanical fixes)
- **Unit tests**: `make test`
- **Integration tests**: `make test-integration`
- **E2E tests**: `make test-e2e` or `make test-e2e-istio`
- **Build**: `make build`

**MUST** prefer `make <target>` over ad-hoc `go test` / `golangci-lint` invocations so that versioned tooling and repo defaults are applied consistently.

## Go version and language features

- **MUST** assume Go version requirements defined by the repo (see README; currently Go `v1.26.0+`).
- **MUST** write idiomatic Go:
  - Prefer clarity over cleverness.
  - Keep functions small and single-purpose.
  - Avoid unnecessary generics; use generics only when they clearly reduce duplication without harming readability.

## Project structure and dependency boundaries

The repo uses a layered package structure under `internal/`. Follow it.

- **MUST** keep new code in the most appropriate existing package (`internal/controller`, `internal/podscanner`, `internal/annotator`, etc.).
- **MUST** avoid cyclic dependencies; prefer “controller → lower layers” flow.
- **SHOULD** keep packages cohesive:
  - `controller`: orchestration and reconcile routing.
  - `podscanner`: cluster reads and detection logic.
  - `annotator`: workload patching/updates.
  - `webhook`: calling Istio injection webhook and parsing.
- **MUST** keep `cmd/main.go` focused on wiring, flags, and manager startup.

## Formatting, naming, and API surface

### Formatting

- **MUST** use `gofmt` (via `make fmt`).
- **MUST NOT** commit code that depends on “manual formatting discipline.”

### Naming

- **MUST** follow Go naming conventions:
  - Package names: short, lower-case, no underscores.
  - Exported identifiers: only when needed by other packages; otherwise keep unexported.
  - Avoid stutter: e.g., `podscanner.Scanner` is better than `podscanner.PodScannerScanner`.
- **MUST** choose names that reflect intent and domain:
  - Use Kubernetes terms consistently: “workload” (Deployment/StatefulSet/DaemonSet), “pod template”, “owner reference”, “namespace label/annotation”, “reconcile request”.

### Public surface area

- **MUST** minimize exported APIs.
- If you export something:
  - **MUST** document it with a complete Go doc comment.
  - **MUST** keep it stable (expect downstream usage).

## Error handling (Go and Kubernetes-specific)

### General rules

- **MUST** wrap errors with context using `%w`:

```go
return fmt.Errorf("parse istio values JSON: %w", err)
```

- **MUST NOT** swallow errors unless the behavior is explicitly desired and safe; when intentionally ignoring an error, you **MUST** justify it in code with a short intent comment.
- **MUST** preserve sentinel error checks when used (e.g., `apierrors.IsNotFound(err)`).

### Controller-runtime reconciliation semantics

- **MUST** treat reconciliation as **idempotent**:
  - Re-running reconcile with the same cluster state must not change anything.
  - Reconcile should converge, not oscillate.
- **MUST** distinguish between:
  - transient errors (requeue is appropriate),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [istio-ecosystem/fortsa](https://github.com/istio-ecosystem/fortsa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
