---
trigger: always_on
description: This document defines the mandatory quality gates, conventions, and task-completion checklist that every AI-generated code contribution to this repository must satisfy. Read it in full before writing any code.
---

# GitHub Copilot Instructions — Corp-Registry Curation Proxy

This document defines the mandatory quality gates, conventions, and task-completion checklist that every AI-generated code contribution to this repository must satisfy. Read it in full before writing any code.

---

## 0. Git Safety Rule (Non-Negotiable)

**NEVER run `git commit` or `git push` unless the user explicitly tells you to.** You may stage files (`git add`), create branches, switch branches, and reset — but committing and pushing require explicit user approval every time.

---

## 1. Quality Gates (Non-Negotiable)

### 1.1 Unit Test Coverage

- Every Go module (`common/`, `pypi-curation/`, `npm-curation/`, and any future ecosystem module) must maintain **≥ 90% statement coverage** at all times.
- Run coverage check before marking any task complete:
  ```bash
  go test -count=1 -race -coverprofile=coverage.out ./...
  go tool cover -func=coverage.out | grep "^total:"
  ```
- PRs that reduce total coverage below 90% are **blocked** in CI.
- New code paths that are genuinely untestable (e.g., `os.Exit` paths, `init()`) must have a documented exemption in the test file via a comment.

### 1.2 Linting

- `golangci-lint run ./...` must produce **zero errors and zero warnings** before any task is complete.
- Enabled linters (configured in `.golangci.yml`):
  - `govet`, `errcheck`, `staticcheck`, `gosimple`, `ineffassign`, `unused`
  - `gocognit` (max cognitive complexity: 15)
  - `goconst` (min string occurrences before requiring constant: 3)
  - `misspell`, `godot`, `gofmt`, `goimports`
- Never suppress linter warnings with `//nolint` without a detailed inline justification.

### 1.3 SonarQube

- Zero **new** bugs, vulnerabilities, security hotspots, or code smells introduced per PR.
- Cognitive complexity per function ≤ 15. Extract helper functions and use early-return patterns to stay within budget.
- No string literal used 3+ times without a package-level constant.
- Test function names must match `^[a-zA-Z0-9]+$` — **no underscores** (use `TestLoadConfigValid`, not `TestLoadConfig_Valid`).
- Security rules: no hardcoded credentials, no `InsecureSkipVerify: true` propagated to production paths, no unvalidated URL redirects.

---

## 2. Documentation Requirements

**All relevant documentation must be updated before a coding task is considered complete.** Documentation is not optional and not deferred to a follow-up PR.

| Change type                   | Documents to update                                                          |
| ----------------------------- | ---------------------------------------------------------------------------- |
| New or changed API endpoint   | `docs/ARCHITECTURE.md` sequence diagrams, `README.md` endpoints table        |
| New ecosystem proxy module    | `docs/ARCHITECTURE.md` component diagram, `README.md` features & quick start |
| New configuration field       | Relevant `config.yaml` examples, `README.md` configuration section           |
| New detection rule type       | `docs/ARCHITECTURE.md`, `README.md` features list                            |
| New CLI flag or env var       | `README.md` usage / environment variables section                            |
| Architecture topology change  | `docs/ARCHITECTURE.md` all affected diagrams                                 |
| User-facing behavioral change | `README.md`                                                                  |
| New Kubernetes manifest       | `docs/ARCHITECTURE.md` deployment section, `README.md`                       |

---

## 3. Go Code Conventions

These conventions are aligned with the SonarQube and linting gates above. Deviating requires explicit justification in the PR description.

### 3.1 Logging

- Use `log/slog` structured logger exclusively.
- Never use bare `log`, `fmt.Println`, `fmt.Printf`, or `os.Stderr.Write` for runtime output.
- Use `slog.With(...)` to attach request-scoped fields (package, version, rule) to log entries.

### 3.2 HTTP Routing

- Use `http.ServeMux` (stdlib) only. No third-party routers (gorilla/mux, chi, echo, gin, etc.).

### 3.3 Metrics

- Use `sync/atomic` counters (`atomic.Int64`) only. No Prometheus client library, no expvar.
- The `/metrics` endpoint serialises these counters to JSON manually.

### 3.4 Configuration

- Use `gopkg.in/yaml.v3` only. No Viper, Koanf, or other config libraries.
- All struct fields must have explicit `yaml:"snake_case_name"` tags.

### 3.5 Error Handling

- Wrap errors with context: `fmt.Errorf("loading config: %w", err)`.
- Never silently discard errors. Use `_` only when the error is provably irrelevant.

### 3.6 Receiver Names

- Consistent per type: `s` for `*Server`, `e` for `*RuleEngine`, `c` for `*Cache`.
- Never change receiver names mid-file.

### 3.7 Architecture Patterns

- Standalone package-level funcs when no receiver state is needed.
- Struct-based walkers for HTML/JSON tree traversal.
- Builder functions (`buildServer`, `applyPortEnvOverride`, `createLogger`) live in `main.go` to keep `main()` thin.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Bluewaves54/Bulwark](https://github.com/Bluewaves54/Bulwark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
