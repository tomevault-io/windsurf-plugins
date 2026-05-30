---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Monorepo Overview

Argus is a black-box red-team testing tool for AI agents. Customers point Argus at any HTTP-spoken or browser-using agent endpoint; Argus produces an attack report mapped to MITRE ATLAS / OWASP LLM Top 10 / NIST AI RMF.

## Services

| Service | Port | Stack | Role |
|---------|------|-------|------|
| `api_service` | 8881 | FastAPI, asyncpg, PostgreSQL | Central REST API: users, R2 storage, `/redteam/*`. Module isolation via `redteam/` + import-linter |
| `orchestrator` | 8081 | FastAPI, Google ADK, litellm | Agent orchestration, SSE streaming, WebSocket agent hub. Probe dispatch + judge harness in `orchestrator/redteam/` |
| `client_agent` | — | websockets, browser-use, Playwright | Edge agent on client/cloud machines. Red-team probe runner in `redteam_runner/` |
| `frontend` | 3000 | Next.js 16, React 19, Zustand | Web UI: marketing site, chat, dashboard |
| `database` | — | Flyway, PostgreSQL | Schema migrations |
| `kubernets` | — | Helm, AKS/k3s | Deployment charts, scripts, YAML manifests |
| `cli` | — | Python (click) | Published `argus-probe` CLI — run / report / list-probes / validate-target |
| `demo_target` | 8000 | FastAPI | Deliberately-vulnerable chatbot used as a public demo target for `argus-probe` |
| `terraform` | — | Terraform | Azure IaC reference (AKS, ACR, PG, Key Vault, B2C apps) |

The three legacy `testing_*` services moved to separate repositories on 2026-05-28:
- `testing_api_service` + `testing_web_fetch_service` → https://github.com/gy15901580825/argus-api-testing
- `testing_web_ui_service` (+ vendored `browser_use`) → https://github.com/gy15901580825/argus-web-ui-testing

The orchestrator still references them over HTTP via `run_api_test` and `run_web_ui_cloud` planner tools; the in-cluster services must be deployed from those repos (their Helm charts ship alongside the code).

## Domain & Environment

### Shared infrastructure

- **Cluster**: Azure AKS, context `<YOUR_AKS>`
- **Container registry**: Azure ACR — `<YOUR_ACR>.azurecr.io/argus/<service>:<tag>`
- **Database server**: Azure Database for PostgreSQL — `<YOUR_PG_SERVER>.postgres.database.azure.com` (`publicNetworkAccess=Disabled`; psql/Flyway must run from in-cluster pods)
- **Ingress**: NGINX Ingress Controller, single LoadBalancer IP `<YOUR_INGRESS_IP>` shared by all hosts
- **Azure Key Vault**: `<YOUR_KEY_VAULT>` is the runtime source-of-truth for `api_service` credentials. K8s Secret `argus-api-service-secret` is auto-synced from KV by External Secrets Operator (ESO) via `ClusterSecretStore: <YOUR_AZURE_KV_STORE>`. ESO authenticates with Workload Identity → UAMI `eso-kv-reader` (federated credential subject `system:serviceaccount:external-secrets:external-secrets`). Other services use literal Helm-rendered Secrets. Operator runbook: `kubernets/DEV_DEPLOYMENT_GUIDE.md` § Secrets management.
- **TLS**: cert-manager + Let's Encrypt (ClusterIssuer: `letsencrypt-prod`), auto-renewed
- **DNS**: Cloudflare proxy (orange cloud) in front of all hosts
- **Auth**: Microsoft Entra External ID (CIAM), SPA app ID `<YOUR_CLIENT_ID>`, in separate tenant `<YOUR_TENANT_ID>`
  - To manage app registrations: `az login --tenant <YOUR_TENANT_ID> --allow-no-subscriptions`
  - SPA `redirectUris` include both prod and dev callbacks: `https://www.example.com/{callback,auth-redirect}`, `https://dev.example.com/{callback,auth-redirect}`, `http://localhost:3000/{callback,auth-redirect}`

### Production

| Dimension | Value |
|---|---|
| URL | `https://www.example.com` |
| Namespace | `default` |
| Database | `argus` |
| Helm release | `argus-<svc>` |
| Image tag | semver (e.g. `1.0.0`) |
| Ingress | `kubernets/ingress-azure.yaml` |
| TLS secret | `argus-tls` |
| Values override | `values-azure.yaml` |
| Secret | `argus-<svc>-secret` (per-service, created by chart from `values.secrets.*`) |

### Dev

| Dimension | Value |
|---|---|
| URL | `https://dev.example.com` |
| Namespace | `dev` |
| Database | `argus_dev` (same PG server, separate logical DB) |
| Helm release | `argus-<svc>-dev` |
| Image tag | `dev-latest` or `dev-<gitsha-short>` |
| Ingress | `kubernets/ingress-azure-dev.yaml` |
| TLS secret | `argus-dev-tls` |
| Values override | `values-azure-dev.yaml` (in addition to `values.yaml` + `values-azure.yaml`) |
| Secrets | `argus-dev-secrets` (shared, covering orchestrator + testing services with all naming-convention aliases) + `argus-api-service-dev-secret` (separate; api-service chart hardcodes `{fullname}-secret` for most keys) |
| Operator manual | `kubernets/DEV_DEPLOYMENT_GUIDE.md` |

**Dev secret-naming gotcha:** Three charts use three different key conventions for the same underlying credentials. `argus-dev-secrets` stores all three sets to satisfy each chart's references:
- orchestrator: `AZURE_API_*` (litellm) + `R2_*`
- testing-{api,web-ui}-service: `AZURE_OPENAI_*` + `CLOUDFLARE_R2_*`
- api-service: `AZURE_OPENAI_*` + `R2_*` (in its own per-service secret because the chart doesn't honor `existingSecretName` for non-DATABASE keys)

## Docker Registry Rules

- **client_agent** → Docker Hub: `<your-gh-user>/client_agent:latest`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gy15901580825/Argus](https://github.com/gy15901580825/Argus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-30 -->
