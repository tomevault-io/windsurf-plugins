---
trigger: always_on
description: This file captures project conventions, open goals, and implementation guidance so
---

# CLAUDE.md — Project Guidance for go-to-production

This file captures project conventions, open goals, and implementation guidance so
that every session (human or AI) stays aligned on what matters.

## Project Overview

A reference implementation that evolves a simple Go todo app into a production-ready
system on Google Cloud Platform. The app code is intentionally minimal — the focus
is on Infrastructure, Security, Observability, and Operational Resilience.

## Repository Layout

```
main.go                  # Entry point — HTTP server, tracing, secrets, DB init
internal/app/app.go      # Core handlers, circuit breaker, retry, metrics, middleware
templates/               # HTML templates (index.html)
static/                  # CSS / JS assets
terraform/               # All GCP infrastructure (GKE, Cloud SQL, IAM, monitoring, …)
k8s/                     # Kubernetes manifests — Kustomize base + region overlays
  base/                  #   Base resources (namespace, service, rollout, HPA, PDB, policies)
  overlays/              #   us-central1 (primary) and us-east1 (secondary) overrides
.github/workflows/       # CI: build-test.yml (build + sign + scan), deploy-pages.yml
scripts/                 # Operational helpers (init-db.sh, generate_cost_chart.py)
test/chaos/              # Chaos / resilience tests (chaos_test.go)
docs/                    # Milestone docs, runbook, strategy, testing guide
```

## Language & Build

- **Go 1.24.4** — standard library `net/http`, no web framework.
- Build: `CGO_ENABLED=0 GOOS=linux go build -o server .`
- Tests: `go test -v ./...` (unit + integration + chaos).
- Docker: multi-stage (`golang:1.25-alpine` → `alpine:latest`).

## Conventions

### Git Tags & Milestones
Every milestone gets a tag: `milestone-NN-slug`.
When starting a new milestone:
1. Ensure the *previous* milestone's work is committed to `main`.
2. `git tag milestone-NN-slug` on the commit that completes it.
3. Push the tag: `git push origin milestone-NN-slug`.

### Documentation
- Each milestone has a doc at `docs/NN_TITLE.md`.
- Runbook entries go in `docs/RUNBOOK.md`.
- Risk matrix lives in `docs/STRATEGY_AND_RISKS.md` — update the Status column
  when a risk is mitigated.

### Terraform
- All infra changes go through `terraform/`.
- State is remote in GCS.
- Multi-region resources are parameterized (`variables.tf` region / secondary_region).

### Kubernetes
- Base manifests in `k8s/base/`, region specifics in `k8s/overlays/`.
- Kustomize for composition. Argo Rollouts for canary. ArgoCD for GitOps sync.
- Gatekeeper policies in `k8s/base/policies/`.

### Security
- No secrets in code or git (gitleaks pre-commit hook).
- Cloud SQL uses IAM auth (no passwords in connection path).
- Binary Authorization enforces signed images.
- Cloud Armor WAF protects the ingress.

### Testing
- Unit tests: `main_test.go` (mocked DB, fast).
- Integration tests: `integration_test.go` (real DB via docker-compose).
- Chaos tests: `test/chaos/chaos_test.go` (circuit breaker, failover).
- CI runs unit + chaos on every push; integration on-demand.

---

## Current State

### Completed Milestones (0 – 12)

| # | Milestone | Tag |
|---|-----------|-----|
| 0 | Baseline | `milestone-00-baseline` |
| 1 | Risk Analysis | `milestone-01-risk-analysis` |
| 2 | Base Infrastructure | `milestone-02-base-infra` |
| 3 | HA & Scalability | `milestone-03-ha-scale` |
| 4 | IAM Auth & Secrets | `milestone-04-iam-auth` |
| 5 | Security Hardening | `milestone-05-security-hardening` |
| 6 | Advanced Deployment | `milestone-06-advanced-deployment` |
| 7 | Observability & Metrics | `milestone-07-observability-metrics` |
| 8 | Robustness & SLOs | `milestone-08-robustness-slos` |
| 9 | Tracing & Polish | `milestone-09-tracing-polish` |
| 10 | GitOps & Automation | `milestone-10-gitops` |
| 11 | Policy & Rollouts | `milestone-11-policy-rollouts` |
| 12 | Supply Chain Security | `milestone-12-supply-chain` |

### In-Progress: Milestone 13 — Multi-Region

Infrastructure deployed (GKE east, DB replica, MCI, ArgoCD). **Remaining:**
- [ ] Verify DB replication lag and connectivity
- [ ] Application config for region-aware DB routing (writes → primary)
- [ ] DNS: point `todo.smig.dev` A record → `34.160.71.244`
- [ ] Verify traffic routes to nearest region
- [ ] Run failover drill (drain us-central1, validate us-east1 serves traffic)
- [ ] Update RUNBOOK.md to mark region failure as mitigated

---

## Open Risks — Implementation Roadmap

Six risks from `docs/STRATEGY_AND_RISKS.md` remain unmitigated.
They are addressed across Milestones 14 – 16 below.

### Milestone 14: Operational Resilience (`milestone-14-operational-resilience`)

> Goal: Close the six highest-priority open risks in the risk matrix.

| Risk (Score) | Deliverable |
|--------------|-------------|
| Quota Exhaustion (6) | Terraform: quota usage monitoring alert policies |
| Sensitive Data Leakage (6) | Go: structured log redaction middleware; policy doc |
| Insider Threat (4) | Terraform: VPC-SC / IAM Recommender / audit log sink |
| Backup Restore Failure (4) | Script + CronJob: automated monthly restore drill |
| Ransomware / Corruption (3) | Terraform: GCS bucket lock + retention policies on backups |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stevemcghee/go-to-production](https://github.com/stevemcghee/go-to-production) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
