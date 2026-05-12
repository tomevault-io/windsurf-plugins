---
trigger: always_on
description: Browser Human-In-The-Loop platform. Workers run Playwright/Chromium to execute Login DSL scripts, with human intervention via Slack/VNC when automation gets stuck (OTP, CAPTCHA, MFA, passwords, magic links, or any custom input).
---

# Tabby (Browser HITL)

Browser Human-In-The-Loop platform. Workers run Playwright/Chromium to execute Login DSL scripts, with human intervention via Slack/VNC when automation gets stuck (OTP, CAPTCHA, MFA, passwords, magic links, or any custom input).

## Build & Test

```bash
pnpm install --frozen-lockfile
pnpm run build
pnpm run test
pnpm run lint
helm lint charts/browser-hitl/
```

**Test with encryption key:** `TENANT_ENCRYPTION_KEY=$(printf '0%.0s' {1..64}) pnpm run test`

**Before committing:** Pre-commit hook runs `lint-staged` + `pnpm run build` + `pnpm run test` automatically via Husky. Commit-msg hook validates conventional commits via commitlint.

## Project Structure

Monorepo (NX + pnpm workspaces):
- `apps/api` — NestJS REST API (port 8000)
- `apps/controller` — Watches sessions, creates/destroys worker pods
- `apps/worker` — Ephemeral Playwright pod, executes Login DSL
- `apps/slack-bot` — NATS subscriber, Slack notifications + human input relay
- `apps/teams-bot` — Microsoft Teams equivalent
- `apps/admin-ui` — Next.js dashboard (port 8000)
- `packages/shared` — Shared types, enums, state machines, DSL types, NATS events
- `charts/browser-hitl` — Helm chart for K8s deployment
- `infra/docker/` — 7 Dockerfiles (api, controller, worker, novnc, slack-bot, teams-bot, admin-ui)
- `infra/tfy/deploy.yaml` — TrueFoundry manifest template (envsubst placeholders)

## Local Development (Kind)

```bash
make kind-create          # one-time: create Kind cluster
make kind-reload-all      # clean + build + docker-build + load + helm upgrade with values-local.yaml
make k8s-port-forward     # forward API (18080), Admin UI (13000), Postgres, Redis, MinIO, NATS
```

All local config (API URL, stream host, service auth, secrets) is in `values-local.yaml` — no manual `kubectl set env` needed.

To enable slack-bot locally, set `slackBot.enabled: true` and add `slackSigningSecret`, `slackAppToken`, `slackBotToken` in `values-local.yaml` secrets section. Remember to also set `slackBot.slackDefaultChannel` to a channel ID.

## Ports (standardized)

ALL services run on port 8000 (API, Admin UI). Controller: 8090, Worker health: 8091. Do NOT use 3000 or 8080 — those are legacy.

## Docker Images

Registry: `ghcr.io/adoptai/tabby/{service}:{tag}`
7 images: api, controller, worker, novnc, slack-bot, teams-bot, admin-ui

## Helm Chart

- Chart: `oci://ghcr.io/adoptai/charts/browser-hitl`
- Lint: `helm lint charts/browser-hitl/`

### Key Helm patterns

- Service names are dynamic: `{{ include "browser-hitl.fullname" . }}-{component}`
- NEVER hardcode service names in values files — use templates with the fullname helper
- ConfigMap URLs (Redis, NATS, MinIO) are auto-generated from fullname helper
- VirtualServices are auto-generated in `templates/virtualservices.yaml`

### Values files

- `values.yaml` — defaults (DO NOT put secrets here)
- `values-local.yaml` — local Kind dev (all config baked in, committed to git)
- `values-staging.yaml` — staging overrides, NOT committed (contains secrets)
- `infra/tfy/deploy.yaml` — CI/CD template, uses `${PLACEHOLDER}` vars substituted by envsubst

## CI/CD (.github/workflows/)

- `ci.yaml` — PR validation: PR title conventional commit check, lint, test, build, security audit, helm lint
- `deploy-staging.yaml` — Push to `dev`: build images → auto-bump chart version → push chart → `tfy apply` → health check
- `deploy-production.yaml` — Push to `main`: reads already-bumped version from Chart.yaml, builds `prod-*` images, same chart version
- Secrets are in GitHub Environments (`staging` / `production`), injected via envsubst into `infra/tfy/deploy.yaml`

### Conventional Commits (enforced)

PR titles MUST follow conventional commits (squash merge makes PR title the commit message):
- `release:` or `feat!:` / `fix!:` → **major** version bump (X+1.0.0)
- `feat:` → minor version bump (0.X+1.0)
- `fix:`, `chore:`, `ci:`, etc. → patch bump (0.0.X+1)
- Enforced by: Husky commit-msg hook (local) + CI PR title check

## Staging Deployment (TrueFoundry)

- Platform: TrueFoundry (wraps ArgoCD)
- Helm release name configured in `infra/tfy/deploy.yaml`
- ArgoCD manages deployment — `helm install/upgrade` directly will CONFLICT
- Istio gateway: `istio-system/tfy-wildcard`

## Architecture Notes

### HITL Flow (Generic Human Input)
Worker hits `request_human_input` DSL step → writes `pending_input_request` to session + signals `AUTH_FAIL` via DB → Controller transitions to `LOGIN_NEEDED` → Creates intervention with `input_request_metadata` + sets baton to `HUMAN_REQUESTED` → Publishes enriched `hitl.started` NATS event (with `intervention_type` + `input_request`) → Slack bot posts dynamic message (buttons adapt to input type) → Human submits value via Slack modal or resolves via VNC → Value stored in Redis (`human_input:{sessionId}:{stepIndex}`, 300s TTL) → Worker polls, receives, acts (fill field / navigate URL / resume) → Health check passes → Session returns to HEALTHY


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adoptai/tabby](https://github.com/adoptai/tabby) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
