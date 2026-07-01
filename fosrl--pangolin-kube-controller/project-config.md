---
trigger: always_on
description: This file provides repository-wide instructions for coding agents working in this repository.
---

# AGENTS.md

## Purpose

This file provides repository-wide instructions for coding agents working in this repository.

The project is a Go-based Kubernetes controller that fetches Pangolin Traefik dynamic configuration from an external endpoint, transforms it into Kubernetes resources, and reconciles those resources into the cluster.

Use this file for global rules. More specific instructions in deeper directories should take precedence for files in those paths.

## Specialized agents and skills

Use the specialized agent profiles in `.github/agents/` when the task clearly matches a role.

- `controller-agent`: controller logic, reconciliation, Pangolin-to-Traefik transformation, Kubernetes runtime behavior, manifests
- `ci-agent`: GitHub Actions, Task-based CI, workflow failures, validation flow
- `docs-agent`: `README.md`, `docs/`, contributor and policy documentation
- `lint-agent`: mechanical lint and formatting fixes only
- `security-agent`: evidence-based security review and minimal remediation
- `test-agent`: unit, integration, E2E, fixtures, and test failure diagnosis

Use repository skills in `.agent/skills/` for repeatable workflows such as controller change review, CI triage, integration-test setup, manifest review, and documentation verification.

This root file defines repository-wide rules. Specialized agents add role-specific guidance. Skills provide reusable workflow instructions, examples, and resources that should be loaded only when relevant.

### Skill usage rules (important)

- Prefer skills over ad-hoc reasoning for repeatable workflows.
- If a matching skill exists in `.agent/skills/`, it MUST be used.
- Do not reimplement workflows already defined in skills.
- Load only relevant skills to avoid unnecessary context usage.

---

## Quick start

Run the smallest relevant set of checks first, then expand only if needed.

### Environment setup

- `go mod download`

### Common validation paths

- Build: `go build ./...`
- Fast tests: `task test`
- Full verification: `task test:crosspkg`
- Integration tests: `task test:integration`
- Coverage merge/report: `task coverage`
- Format: `task fmt`
- Lint: `task lint`
- CI reproduction: `task ci`

### Notes

- Integration tests require `setup-envtest`.
- Prefer targeted validation for small changes, and broader validation for cross-cutting changes.
- Do not claim a command passed unless you actually ran it.

---

## Repository snapshot

### Tech stack

- Language: Go
- Module: `pangolin-kube-controller`
- Primary domain: Kubernetes controller / operator-style reconciliation
- Main concerns: config fetch, transform, apply, garbage collection, observability, release automation
- Deployment model: Kubernetes-focused controller runtime; this repository is intended to run as a Kubernetes workload rather than a general standalone service.
- External config source: Pangolin.
- Reconciled output: Traefik-related Kubernetes resources.

### Key directories

- `cmd/controller/`: main controller entrypoint
- `cmd/healthcheck/`: healthcheck entrypoint
- `internal/controller/`: reconciliation loop, fetch, apply orchestration, backoff, leader election, readiness
- `internal/apply/`: server-side apply helpers and resource application logic
- `internal/config/`: environment/file config loading, defaults, normalization
- `internal/httpserver/`: `/healthz`, `/readyz`, `/metrics`, optional TLS
- `internal/kube/`: Kubernetes client and label/resource helpers
- `internal/observability/`: logging, Prometheus metrics, OpenTelemetry metrics
- `internal/reconcile/`: garbage collection
- `internal/transform/`: routing, protocol conversion, sanitization, config transformation
- `test/integration/`: integration tests
- `test/e2e/`: offline E2E tests
- `docs/`: project and process documentation
- `hack/`: scripts, task includes, and helper tools
- `.github/workflows/`: CI/CD pipelines
- `.github/agents/`: GitHub custom agent role files

### Runtime model

The controller typically:

1. loads configuration from env and/or file
2. polls the Pangolin config endpoint
3. detects whether the config changed
4. transforms config into Kubernetes-native objects
5. applies those resources using server-side apply
6. garbage-collects no-longer-managed resources
7. exposes health and metrics endpoints
8. optionally participates in leader election

---

## Golden rules

These apply to every agent, regardless of task.

- Controller behavior must remain idempotent and safe under retries.
- Verify before claiming. Never invent commands, outputs, files, behavior, or CI results.
- Keep diffs small. Avoid unrelated formatting, renames, or refactors.
- Preserve behavior unless the task explicitly asks for behavior changes.
- Preserve security posture. Do not weaken auth, permissions, validation, TLS handling, secret handling, or logging hygiene.
- Preserve least privilege in CI and deployment automation.
- Update tests when behavior changes.
- Update docs when user-facing behavior, commands, configuration, or workflows change.
- Prefer targeted checks first, then broader checks as needed.
- Never commit secrets, tokens, kubeconfigs, certificates, or private credentials.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fosrl/pangolin-kube-controller](https://github.com/fosrl/pangolin-kube-controller) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
