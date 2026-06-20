---
trigger: always_on
description: Discover a company's tech stack, observability stack, infrastructure scale, observability costs, and pain points across all environments. Detection is agent-driven; all volume and cost figures are computed by deterministic collector scripts with raw API evidence, merged into a single verifiable executive-level HTML report.
---


# Infrastructure & Observability Discovery

This skill guides the agent through a systematic discovery of a company's tech stack, observability stack, infrastructure scale, observability costs, and operational pain points. The output is one self-contained HTML report: an executive summary up top (scannable in under 2 minutes), collapsible per-tool deep dives below, and a provenance appendix linking every measured figure to the raw API response that produced it.

For install instructions, see [README.md](README.md).

## Division of labor (the core rule)

```
You (the agent)                      Collector scripts (deterministic)
────────────────────────             ──────────────────────────────────
detect environments & tools     →    collectors/<tool>/collect.py
ask qualitative questions       →    compute every volume/cost figure
write context.json              →    write summary.json + evidence/
run the report generator        →    report/generate_report.py writes the HTML
```

**You never compute a reported figure.** No improvised PromQL, no unit conversions, no arithmetic on API responses, no editing the report HTML. Every number in the report comes from a collector's `summary.json`; everything you contribute goes into `context.json` and is rendered as qualitative or "reported — not verified". If a collector fails, the answer is a documented gap — never a substituted estimate.

## Principles

1. **Non-destructive only.** Never modify, delete, or write to any system. All operations are read-only.
2. **Deterministic figures.** Volume and cost numbers come only from collector scripts, which capture the raw API responses they used (evidence files).
3. **Observability costs only.** Discover observability spend (CloudWatch, Datadog, self-hosted stack footprint, licenses). Never query or report account-wide cloud costs. For everything that isn't observability, record broad inventory only (counts, instance families, telemetry-relevant managed services).
4. **Never silently omit.** Anything that couldn't be collected appears in the report's Coverage & Gaps section with the reason and a remediation.
5. **Rate-limited.** Wait 1-2 seconds between ad-hoc API calls; collectors throttle themselves.
6. **Ask when uncertain.** Information that can't be discovered programmatically is asked of the user and recorded as user-reported.
7. **Plan first.** Always present the discovery plan and get user approval before executing.
8. **Multi-environment aware.** Discover all environments (dev, staging, prod, etc.) separately.

## Execution Flow

### Phase 0: Present Plan

Before doing anything, present this plan to the user and ask for approval:

```
I'll discover your infrastructure and observability setup. Here's my plan:

1. **Environment Detection** — Identify all environments (cloud accounts, k8s clusters, regions)
2. **Tech Stack Discovery** — Languages, frameworks, databases, message queues (broad inventory)
3. **Infrastructure Inventory** — Compute scale and telemetry-relevant managed services (counts only)
4. **Observability Stack Discovery** — Monitoring, logging, tracing, alerting tools
5. **Scale & Cost Measurement** — For each detected observability tool I'll run a read-only
   collector script that queries its usage/billing APIs and saves the raw responses as evidence.
   Costs are scoped to observability spend only — I will not query your overall cloud bill.
6. **Pain Points** — Observability-specific: alert fatigue, coverage gaps, tool sprawl, cost
7. **Report** — A single HTML report; every measured number links to its raw API evidence,
   and anything that couldn't be collected is listed with the reason.

All operations are read-only. Collector scripts run locally via `uv run`; nothing is uploaded.
Shall I proceed?
```

Wait for user confirmation before continuing.

### Phase 0.5: Prerequisites

Collector scripts are PEP 723 Python scripts run with `uv`:

```bash
uv --version 2>/dev/null
python3 --version 2>/dev/null   # needs >= 3.11
```

- If `uv` is missing, offer to install it (`curl -LsSf https://astral.sh/uv/install.sh | sh`) or via `pipx install uv` / `brew install uv`. Ask before installing anything.
- If the user declines or the machine is airgapped, continue in **degraded mode**: skip all collectors, record each skipped one in `context.json.skipped_collectors`, and still generate the report — the volumes/costs will appear as gaps.

### Phase 1: Environment Detection

Discover all environments by checking these sources in order. Stop each check after 5 seconds if no response.

**Before starting Phase 1, cache commonly used Kubernetes data to avoid redundant API calls:**

```bash
kubectl get pods -A -o json 2>/dev/null > /tmp/discovery-pods.json
kubectl get crd 2>/dev/null > /tmp/discovery-crds.txt
```

Use `/tmp/discovery-pods.json` and `/tmp/discovery-crds.txt` for all subsequent pod and CRD queries instead of re-fetching from the API.

#### 1.1 Kubernetes Clusters

```bash
kubectl config get-contexts 2>/dev/null

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oodle-ai/discovery-agent-skills](https://github.com/oodle-ai/discovery-agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
