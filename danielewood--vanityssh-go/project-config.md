---
trigger: always_on
description: Generates ED25519 SSH key pairs at high speed and matches the resulting public
---

# CLAUDE.md — vanityssh-go

## Rule Severity

**MUST** rules are enforced by CI/pre-commit; **SHOULD** rules are strong
recommendations; **CAN** rules are allowed without extra approval. Stable IDs
(e.g., **ERR-1**, **CC-2**) enable precise code-review comments and automated
policy checks. Keep IDs stable; deprecate with notes instead of renumbering.

---

## 0 — Project Overview

Go module: `github.com/danielewood/vanityssh-go`
Go version: 1.24+
License: MIT
Binary: `vanityssh`

Generates ED25519 SSH key pairs at high speed and matches the resulting public
keys (or SHA256 fingerprints) against a user-supplied regex. First match writes
`id_ed25519` / `id_ed25519.pub` and exits (unless `--continuous`).

### Key behaviors

- Spawns N goroutines (default: `runtime.NumCPU()`) generating keys in tight
  loops
- First match writes key files and exits (unless `--continuous`)
- Supports matching against base64 public key or SHA256 fingerprint
  (`--fingerprint`)
- Benchmark mode for pure key generation throughput

---

## 1 — Before Coding

- **BP-1 (MUST)** Ask clarifying questions for ambiguous requirements.
- **BP-2 (MUST)** Draft and confirm an approach (API shape, data flow, failure
  modes) before writing code.
- **BP-3 (SHOULD)** When >2 approaches exist, list pros/cons and rationale.
- **BP-4 (SHOULD)** Define testing strategy and observability signals up front.

---

## 2 — Package Structure

```text
main.go                 # Thin entry point — sets version, calls cmd.Execute()
cmd/
  root.go               # Cobra CLI: flags, validation, worker orchestration
keygen/
  keygen.go             # Hot-path key generation worker, Result type
  keygen_test.go        # Unit tests for generation, matching, cancellation
display/
  display.go            # TTY detection, scroll-region status bar, formatting
  display_test.go       # Tests for formatting and TTY helpers
```

---

## 3 — Modules & Dependencies

- **MD-1 (SHOULD)** Prefer stdlib; introduce deps only with clear payoff.
- **MD-2 (SHOULD)** Run `govulncheck` before adding or updating dependencies.

---

## 4 — Code Style

- **CS-1 (MUST)** Enforce `gofmt`, `go vet`, `goimports` before committing.
- **CS-2 (MUST)** Avoid stutter in names: `package kv; type Store` (not
  `KVStore` in `kv`).
- **CS-3 (SHOULD)** Small interfaces near consumers; prefer composition over
  inheritance.
- **CS-4 (SHOULD)** Avoid reflection on hot paths; prefer generics when it
  clarifies and speeds.
- **CS-5 (MUST)** Use input structs for functions receiving more than 2
  arguments. `context.Context` stays outside input structs.
- **CS-6 (SHOULD)** Declare function input structs before the function consuming
  them.

### Go version

Target the latest stable Go release. Use modern stdlib features freely:
`slices` package, `min`/`max` builtins, range-over-integers where it simplifies
iteration.

### Formatting and imports

- Two import groups: stdlib, then third-party. Alphabetical within each group.
- No blank lines within an import group.

### Naming

- Exported functions: doc comment required (godoc style). No exceptions.
- Unexported functions: doc comment if the purpose isn't obvious from the name.
- Error variables: `errFoo` (unexported), `ErrFoo` (exported).
- Test helpers: always call `t.Helper()`.

### Philosophy

- Boring and readable over clever and terse.
- DRY: extract helpers when logic repeats.
- No premature abstractions — keep code straightforward.
- Consistency with existing patterns trumps personal preference.

---

## 5 — Errors

- **ERR-1 (MUST)** Wrap with `%w` and context:
  `fmt.Errorf("compile regex %q: %w", pattern, err)`.
- **ERR-2 (MUST)** Use `errors.Is`/`errors.As` for control flow; no string
  matching.
- **ERR-3 (SHOULD)** Define sentinel errors in the package; document behavior.
- **ERR-4 (MUST)** Lowercase error strings, no trailing punctuation.
- **ERR-5 (MUST)** Never silently ignore errors.
- **ERR-6 (MUST)** Fail fast with descriptive messages.

---

## 6 — Concurrency

- **CC-1 (MUST)** The **sender** closes channels; receivers never close.
- **CC-2 (MUST)** Tie goroutine lifetime to a `context.Context`; prevent leaks.
- **CC-3 (MUST)** Protect shared state with `sync.Mutex`/`atomic`; no "probably
  safe" races. `global_counter` must use `atomic` operations.
- **CC-4 (SHOULD)** Use `errgroup` for fan-out work; cancel on first error.
- **CC-5 (CAN)** Prefer buffered channels only with rationale
  (throughput/back-pressure).

---

## 7 — Contexts

- **CTX-1 (MUST)** If a function takes `ctx context.Context` it must be the
  first parameter; never store ctx in structs.
- **CTX-2 (MUST)** Propagate non-nil `ctx`; honor `Done`/deadlines/timeouts.

---

## 8 — Testing

### Requirements

- **T-1 (MUST)** All tests must pass before committing. Run `go test ./...`,
  `go vet ./...`, and `golangci-lint run`.
- **T-2 (MUST)** Table-driven tests with descriptive subtest names as the
  default pattern.
- **T-3 (MUST)** Run `-race` in CI; add `t.Cleanup` for teardown.
- **T-4 (SHOULD)** Mark safe tests with `t.Parallel()`.
- **T-5 (SHOULD)** Tests use stdlib `testing` only (no testify/gomock).

```sh
go test ./...          # Run all tests
go build ./...         # Verify compilation
go vet ./...           # Static analysis

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danielewood/vanityssh-go](https://github.com/danielewood/vanityssh-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
