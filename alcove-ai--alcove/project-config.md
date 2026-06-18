---
trigger: always_on
description: Alcove runs AI coding agents (Claude Code) in ephemeral, network-isolated
---

# Alcove — Sandboxed AI Coding Agents on OpenShift/Kubernetes

## What This Is

Alcove runs AI coding agents (Claude Code) in ephemeral, network-isolated
containers. Each session gets a fresh container, a scoped authorization proxy, and
a complete session transcript. No persistent state crosses session boundaries.

**Language:** Go 1.25 | **License:** Apache-2.0

## Components

| Name | Role | Binary | k8s Resource |
|------|------|--------|-------------|
| **Bridge** | Controller, REST API, dashboard, scheduler, agent repo syncer | `cmd/bridge` | Deployment |
| **Skiff** | Ephemeral Claude Code worker | `cmd/skiff-init` | Job / `podman run --rm` |
| **Gate** | Auth proxy sidecar (token swap, LLM proxy, SCM proxy, scope enforcement) | `cmd/gate` | Sidecar in Skiff pod |
| **Shim** | Execution sidecar injected into dev containers (`GET /healthz`, `POST /exec` with NDJSON streaming) | `cmd/shim` | Sidecar in dev container |
| **Hail** | Message bus (NATS) | external | Deployment |
| **Ledger** | Session store (PostgreSQL) | external | Deployment + PVC |

## Architecture

```
Bridge → Hail (NATS) → Skiff Pod [skiff container + gate sidecar] → Gate → External Services
                                ↕ /workspace volume (optional)              → Ledger (PostgreSQL)
                        Dev Container [project image + shim]
```

- Skiff pods are ephemeral: one session, one container, then destroyed
- Gate is a sidecar (shares network namespace with Skiff)
- Gate proxies ALL external traffic: MITM TLS interception for service domains (GitHub, GitLab, Jira) and `/v1/` endpoint for LLM API calls (Skiff has no real credentials)
- Optional dev container runs alongside Skiff with a shared `/workspace` volume, enabling agents to build/test code in project-specific environments; the shim binary is baked into the dev container image (built with `make build-dev`); a simple shell entrypoint starts PostgreSQL, NATS, and the shim without requiring a process supervisor
- On OpenShift, a static `alcove-allow-internal` NetworkPolicy restricts egress (per-task NetworkPolicy is disabled due to OVN-Kubernetes DNS resolution issues); dual-network isolation (`--internal` flag) on podman
- k3s is supported for local Kubernetes development (`make k3s-setup && make k3s-up && make k3s-watch`); Bridge runs on the host, NATS+PostgreSQL run as k8s pods with port-forwards, Skiff Jobs are dispatched into the same k3s cluster

## Design Documents

Read these for full context:

1. `docs/design/implementation-status.md` — **START HERE** — current state, what works, what's next
2. `docs/glossary.md` — **TERMINOLOGY** — canonical definitions for workflow, session, task, etc.
3. `docs/design/architecture.md` — component design, deployment diagrams, network isolation, roadmap
4. `docs/design/architecture-decisions.md` — 24 resolved decisions, CLI design, config format, repo layout
5. `docs/design/problem-statement.md` — why ephemeral agents
6. `docs/design/credential-management.md` — credential storage, encryption, OAuth2 token flow
7. `docs/design/auth-backends.md` — auth backend design (memory, postgres, rh-identity)
8. `docs/design/gate-scm-authorization.md` — SCM MITM proxy, operation taxonomy, security model

## Skills (Slash Commands)

These are available as `/command` in Claude Code sessions. Use them instead of
doing these tasks manually.

| Command | When to Use |
|---------|-------------|
| `/dev-up` | First-time setup or full reset of local dev environment (wipes database) |
| `/dev-restart` | Rebuild and restart Bridge after code changes (preserves database) |
| `/release` | Trigger the automated release pipeline (creates changelog, tags, builds images) |
| `/deploy-staging` | Audit a release for safety and deploy to OpenShift staging via app-interface |

Skill definitions live in `.claude/skills/` — read them for full details.

## Quick Commands

```bash
# Primary dev workflow — hot-reload with full session dispatch
make watch                    # Builds images, starts NATS+PostgreSQL, runs Bridge via Air
                              # Save a .go file → Air rebuilds → Bridge restarts → sessions work
make down                     # Stop everything (Bridge + NATS + PostgreSQL)

# First-time setup or database wipe (then switch to make watch)
make up                       # Build binaries + images, start containerized Bridge + infra
                              # Requires follow-up curl commands to seed credentials (see dev-up skill)

# Build
make build                    # Build all Go binaries to bin/
make build-images             # Build container images with podman (smart rebuild via stamps)
make build-tooling            # Build heavy skiff-tooling base image (only when tools change)
make test                     # Run tests

# Infrastructure
make dev-infra                # Start only NATS + PostgreSQL on podman
make dev-up                   # Start full containerized environment
make dev-down                 # Stop everything
make dev-reset                # Stop + remove volumes (clean slate)

# k3s (Kubernetes backend) — test k8s runtime locally
# Requires sudo for k3s install and image import. Stop podman infra first: make down

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alcove-ai/alcove](https://github.com/alcove-ai/alcove) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
