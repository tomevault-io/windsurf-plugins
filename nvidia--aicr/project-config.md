---
trigger: always_on
description: This file contains extended technical documentation for GitHub Copilot. For core rules and patterns, see `AGENTS.md`.
---

# Copilot Instructions for NVIDIA AI Cluster Runtime (AICR)

This file contains extended technical documentation for GitHub Copilot. For core rules and patterns, see `AGENTS.md`.

## Critical Rules (Always Apply)

**Code Quality (Non-Negotiable):**
- Tests must pass with race detector (`make test`)
- Never disable tests to make CI green (including "temporary" skips)
- Use structured errors from `pkg/errors` with error codes (never `fmt.Errorf`)
- Never return bare `err` — always wrap with `errors.Wrap(code, "context message", err)`
- Context with timeouts for all I/O operations (collectors: 10s, handlers: 30s)
- Never use `context.Background()` in I/O methods — use `context.WithTimeout()` instead
- Always use `slog` for logging — never `fmt.Println`/`fmt.Printf` in production code
- Use named constants from `pkg/defaults` — no magic timeout/limit literals
- Check `ctx.Done()` in loops and long-running operations
- Stop after 3 failed attempts at same fix → reassess approach

**Error Wrapping Discipline:**
```go
// REQUIRED: Always wrap errors with context
if err := doThing(); err != nil {
    return errors.Wrap(errors.ErrCodeInternal, "failed to do thing", err)
}

// EXCEPTION: Don't re-wrap when inner error already has correct code
content, err := readContent(ctx, path) // already returns pkg/errors with proper code
if err != nil {
    return err // preserve inner error's code
}

// EXCEPTION: K8s helpers and Close() returns don't need wrapping
return k8s.IgnoreNotFound(err)
```

**Context Propagation:**
```go
// BAD: unbounded context in I/O method
func (r *Reader) Read(url string) ([]byte, error) {
    return r.ReadWithContext(context.Background(), url) // can hang forever
}

// GOOD: timeout-bounded context
func (r *Reader) Read(url string) ([]byte, error) {
    ctx, cancel := context.WithTimeout(context.Background(), r.TotalTimeout)
    defer cancel()
    return r.ReadWithContext(ctx, url)
}

// context.Background() is OK ONLY for: cleanup defers, graceful shutdown, test setup
```

**Git Configuration:**
- Commit to `main` branch (not `master`)
- Do NOT add `Co-Authored-By` lines (organization policy)
- Use cryptographic commit signing: `git commit -S -m "message"` (use `-s` only when DCO sign-off is required for non-member contributions)

**Decision Framework:**
Choose solutions based on: testability, readability, consistency, simplicity, reversibility

**Kubernetes Patterns:**
- Use watch API instead of polling loops for efficiency
- Use shared utilities from `pkg/k8s/pod` (WaitForJobCompletion, StreamLogs, GetPodLogs, WaitForPodReady, ParseConfigMapURI)
- Never reimplement Job/Pod operations — always check `pkg/k8s/pod` first

**Test Isolation:**
- Always use `--no-cluster` flag in E2E tests (chainsaw, tools/e2e)
- Always use `validator.WithNoCluster(true)` in unit tests
- Never connect to production clusters from tests
- Test mode skips RBAC creation and Job deployment

**Configuration Options:**
- Use pointer pattern for optional config (nil = not set, &value = set)
- Avoid boolean flags to track whether option was set — pointers make intent clear

---

## Project Context

NVIDIA AICR provides validated GPU-accelerated Kubernetes configurations through a four-stage workflow:

1. **Snapshot** → Capture system state (OS, kernel, K8s, GPU)
2. **Recipe** → Generate optimized config from captured data or query parameters
3. **Validate** → Check recipe constraints against actual cluster measurements
4. **Bundle** → Create deployment artifacts (Helm values, manifests, scripts)

**Core Components:**
- **CLI (`aicr`)**: All four stages (snapshot/recipe/validate/bundle)
- **API Server (`aicrd`)**: Recipe generation and bundle creation via REST API
- **Agent**: Kubernetes Job for automated cluster snapshots → ConfigMaps
- **Bundlers**: Plugin-based artifact generators — one per component in `recipes/registry.yaml`
- **Deployers**: GitOps integration providers (helm, argocd) with deployment ordering

**Tech Stack:** Go 1.26, Kubernetes 1.33+, golangci-lint v2.10.1, Container images via Ko

**Package Architecture (Critical Principle):**
- **User Interaction Packages** (`pkg/cli`, `pkg/api`): Focus solely on capturing user intent, validating input, and formatting output. No business logic.
- **Functional Packages** (`pkg/oci`, `pkg/bundler`, `pkg/recipe`, `pkg/collector`, `pkg/validator`): Self-contained, reusable business logic. Should be usable independently without CLI/API.
- **Shared Utilities** (`pkg/k8s/pod`, `pkg/defaults`, `pkg/errors`): Common functionality reused across multiple packages. Avoid duplication.
- **Example**: OCI packaging logic lives in `pkg/oci` (not `pkg/cli`), so both CLI and API can use it. Job/Pod operations live in `pkg/k8s/pod` so both agent packages can use them.

---

## Common Tasks (Start Here)

### I Need To: Add GPU Support for New Hardware

1. **Add collector** in `pkg/collector/gpu/`:
   - Implement `Collector` interface
   - Add factory method in `factory.go`
   - Write table-driven tests with mocks

2. **Update recipe data** in `recipes/`:
   - Add base measurements in `overlays/base.yaml`
   - Create overlay files in `overlays/` with criteria matching

3. **Test workflow**:
   ```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA/aicr](https://github.com/NVIDIA/aicr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
