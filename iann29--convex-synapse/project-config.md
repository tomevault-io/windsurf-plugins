---
trigger: always_on
description: Project-level instructions for Claude Code when working in this repo.
---

# CLAUDE.md

Project-level instructions for Claude Code when working in this repo.
A short, opinionated guide so a fresh Claude session can be productive
immediately. Keep it under ~250 lines.

## What this project is

**Synapse** is an open-source control plane for self-hosted Convex deployments
— it replicates the Cloud's "Big Brain" management layer (teams, projects,
multi-deployment, audit log, CLI auth) on infrastructure the user controls.

Big Brain itself is closed-source; we re-implement (a subset of) the public
[OpenAPI v1 spec](https://github.com/get-convex/convex-backend/blob/main/npm-packages/dashboard/dashboard-management-openapi.json)
that the Convex Cloud dashboard talks to. The dashboard fork in `dashboard/`
talks to Synapse with the same shape Cloud uses.

## Repo layout

| Path | What's there |
|---|---|
| `synapse/cmd/server/main.go` | Entrypoint. Wires DB, JWT, docker, health worker, provisioner worker, optional reverse proxy, optional crypto box for HA |
| `synapse/internal/api/` | HTTP handlers (one file per resource: auth, teams, projects, deployments, invites, access_tokens, audit_log, health) |
| `synapse/internal/audit/` | Best-effort writer (`Record`) hooked into every mutating handler |
| `synapse/internal/auth/` | Password hash, JWT issuer, opaque-token helpers, request context |
| `synapse/internal/crypto/` | AES-256-GCM envelope (`SecretBox`) used for `deployment_storage` Postgres URL + S3 keys. v0.5+, opt-in via `SYNAPSE_STORAGE_KEY` |
| `synapse/internal/db/` | pgx pool + embedded SQL migrations + `WithRetryOnUniqueViolation` + `WithTryAdvisoryLock` helpers |
| `synapse/internal/docker/` | Docker SDK wrapper. `Provision[Replica]/Destroy[Replica]/Status[Replica]/Restart[Replica]/GenerateAdminKey`. v0.5 split single-replica vs HA paths via `DeploymentSpec.HAReplica` + `Storage` |
| `synapse/internal/health/` | Periodic worker that reconciles `deployment_replicas.status` with Docker reality, then rolls up to `deployments.status`. Optional auto-restart |
| `synapse/internal/middleware/` | chi middleware (auth, logging, CORS) |
| `synapse/internal/models/` | Domain types — JSON tags match OpenAPI v1. v0.5 added `Deployment.HAEnabled/ReplicaCount`, `DeploymentReplica`, `DeploymentStorage` |
| `synapse/internal/provisioner/` | Persistent job queue; `Worker` runs N parallel goroutines pulling via `SELECT FOR UPDATE SKIP LOCKED`. v0.5 reads `replica_id` and decrypts `deployment_storage` for HA jobs |
| `synapse/internal/proxy/` | Optional `/d/{name}/*` reverse proxy. v0.5 returns multi-replica address list and fails over on connection error |
| `synapse/internal/test/` | Integration test suite (`Setup(t)` / `SetupHA(t)` harness + per-resource `_test.go`). Package: `synapsetest` |
| `synapse/internal/db/migrations/` | `go:embed`'d SQL migrations applied at startup. v1.0.3+: 9 migrations (init, jobs, adopted, replicas, replica_id on jobs, upgrade_to_ha kind, app token scope, project_members RBAC, deploy_keys repurpose) |
| `dashboard/` | Next.js 16 + Tailwind 4 dashboard. Real pages, not a placeholder. HA toggle + `HA ×N` badge on deployments since v0.5 |
| `dashboard/tests/` | Playwright e2e (24 specs) — runs against live compose stack |
| `dashboard/app/setup/` | v0.6.3 first-run wizard. `/login` redirects here when `/v1/install_status` reports `firstRun=true` |
| `setup.sh` | v0.6 auto-installer entry point. `main()` wrapper for curl-pipe-shell safety, bootstrap re-exec under `curl \| bash`, ERR/EXIT traps, flock single-instance, full CLI flag surface (install + lifecycle) |
| `installer/lib/` | Pure-bash detection helpers (detect:: namespace) — OS, arch, pkg manager, sudo, has_*, disk/RAM, public_ip |
| `installer/install/` | Phase scripts the orchestrator composes (preflight, secrets, caddy, compose, verify, ui) + `lifecycle.sh` (upgrade / backup / restore / uninstall / logs / status) + `wizard.sh` (interactive Q&A walkthrough fired when no mode flags passed) + `updater.sh` (phase_install_updater, drops the v1.1.0+ self-update systemd daemon) |
| `installer/updater/` | Self-update daemon bundle — `synapse-updater` (Python 3 HTTP server on a unix socket) + `synapse-updater.service` (systemd unit). Lives on the host, not in docker compose, so it survives the upgrades it orchestrates |
| `installer/templates/` | env.tmpl + caddy.fragment + caddy.standalone — rendered with `{{KEY}}` substitution from exported env vars |
| `installer/test/` | bats unit tests (327 cases) + Dockerfile that adds jq+curl to bats/bats:latest |
| `docs/` | ARCHITECTURE, ROADMAP, QUICKSTART, API, DESIGN, V0_5_PLAN, V0_6_INSTALLER_PLAN, HA_TESTING, PRODUCTION, HANDOFF |
| `docker-compose.yml` | Local dev stack: postgres + synapse + dashboard. Optional `ha` profile (backend-postgres + minio) and `caddy` profile (TLS reverse proxy) |
| `.env.example` | Every config var the backend reads, including the `SYNAPSE_HA_*` and `SYNAPSE_BACKEND_*` knobs |
| `.vps/` | **gitignored** — synapse-test VPS credentials + private SSH key. NEVER commit |

## Common commands

```bash
# Bring up the full stack
docker compose up -d

# Rebuild + restart synapse only
docker compose build synapse && docker compose up -d synapse

# Reset DB tables (TRUNCATE — keeps schema)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Iann29/convex-synapse](https://github.com/Iann29/convex-synapse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
