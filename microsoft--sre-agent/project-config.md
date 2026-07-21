---
trigger: always_on
description: Guidance for any AI agent (GitHub Copilot CLI/IDE, Claude Code, Cursor, Codex, etc.) working in this repo.
---

# AGENTS.md — SRE Zava Demo

Guidance for any AI agent (GitHub Copilot CLI/IDE, Claude Code, Cursor, Codex, etc.) working in this repo.

Azure SRE Agent demo — AKS + PostgreSQL e-commerce app with break/fix scenarios and autonomous incident response.

## Getting Started

When a user clones this repo, guide them through setup:
1. Check prerequisites: `az`, `azd`, `pwsh` — install any missing. (`kubectl` is **not** required on the operator workstation: the AKS cluster is private, and operator in-cluster operations run through `az aks command invoke`; the deployed SRE Agent uses native `kubectl` — authenticated by its managed identity via `kubelogin` — for in-cluster work.)
2. Run `azd up` — pick their default subscription, use `swedencentral` region
3. After deploy completes, run `scripts/setup-sre-agent.ps1` to upload the knowledge file and verify the Bicep-deployed agent (the agent itself, connectors, skills, response plans, mode, incident binding are all already provisioned by Bicep — this script only handles the data-plane KB upload and a verification readout)
4. Open the storefront in browser to verify it works
5. Run a break scenario to demonstrate the SRE Agent

Use browser tools (Playwright MCP or Chrome DevTools) to visually verify the storefront after deployment.

## Non-Obvious Things (repo / IaC author concerns)

These are gotchas for someone editing this repo's IaC or Bicep — they're *not* runtime-incident knowledge. The runtime knowledge the SRE Agent itself needs (network reachability, ECONNREFUSED vs ETIMEDOUT, `bin/run-sql.js` invocation pattern, NSG-vs-NetworkPolicy, App Insights filter requirements, etc.) lives in [`sre-config/knowledge-base/zava-architecture.md`](sre-config/knowledge-base/zava-architecture.md), which gets uploaded to the agent on deploy.

- **K8s manifests** use `${VAR}` placeholders — substituted by `post-provision.ps1`, not Helm/Kustomize.
- **No `azd deploy api`** — there's no `services:` block in `azure.yaml`. Container images are built by `post-provision.ps1` via `az acr build`. To iterate: `az acr build --registry $acr --image zava-api:latest ./src/api` then `az aks command invoke … kubectl rollout restart deployment …`.
- **Activity log alerts default to Sev4** because `Microsoft.Insights/activityLogAlerts` rules don't expose a severity field for the categories we use (Administrative). Response plans must include `Sev4` in `priorities[]` (this repo includes all severities) or activity-log-driven incidents won't match.
- **Response plans are granular: 3 known-good filters + 1 unknown bucket.** `infra/modules/sre-agent.bicep` defines four `incidentFilters`, routed by `titleContains` (handlingAgent `default`, so the agent picks a skill by description): `zava-database` (`postgres`), `zava-performance` (`query-slow`), `zava-application` (`http-5xx`) — all autonomous — plus `zava-unknown`, a catch-all (`titleContainsAny ['Zava','postgres']` + `titleNotContains ['postgres','query-slow','http-5xx']`) that runs in **Review** mode with deep investigation for novel incidents. There is NO documented precedence when multiple filters match, so the buckets are kept non-overlapping via `titleNotContains`; the unknown bucket is bounded to demo-named alerts so it can't sweep in subscription noise. Exercise it with the disabled `Zava-unknown-test` alert.
- **One primary dispatching alert per failure domain (de-noised).** A single root cause used to fire up to 5 alerts → 5 threads. Now there are **three enabled** dispatching alerts: `postgres-unreachable` (DB stop OR network partition — see the next bullet), `Zava-products-query-slow` (perf), `Zava-http-5xx-errors` (app). The 5xx alert fires purely on the 5xx count — **no DB self-suppression**, and agent **merge is off on all plans**, so we don't dedupe: a DB outage that also returns 5xx opens its own app thread alongside the `postgres-unreachable` thread (each real symptom surfaces its own investigation). The `performance-incidents` skill corroborates from the underlying metrics, which flow independently of any alert — `cpu_percent` (PG `AllMetrics` diagnostic → `AzureMetrics`) and the `zava.products.category.query.duration_ms` custom metric (OTel → `AppMetrics`); we deploy only the dispatching alerts the demo actually fires, not disabled metric-alert examples. The NSG-change activity alerts (noise from platform-managed NSG changes) and the redundant `postgres-server-stopped` activity alert were removed.
- **Dispatching scheduled-query alerts use `evaluationFrequency: PT5M`, not `PT1M`.** Verified live across all three scenarios: at 1-minute evaluation the alert fires and is *acknowledged* but the SRE agent does **not** open an autonomous investigation thread; at 5-minute evaluation it dispatches and remediates end-to-end. `postgres-unreachable`, `Zava-products-query-slow`, and `Zava-http-5xx-errors` are therefore all `PT5M`/`PT5M` (eval/window). The DB/perf signals are unambiguous, so firing ~3-5 min after the break (vs ~2 min) is well within demo tolerance. Keep any new dispatching alert at `PT5M`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/sre-agent](https://github.com/microsoft/sre-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
