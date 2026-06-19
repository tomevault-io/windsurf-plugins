---
trigger: always_on
description: Analyzes Azure resource group costs, identifies optimization opportunities, and generates professional FinOps reports. This skill should be used when the user wants to analyze Azure resource group spending, find orphaned resources, get cost optimization recommendations, or generate cost analysis reports. Invoked with a resource group name and optional region.
---


# Azure Resource Group Cost Analyzer

Analyze an Azure resource group to produce a comprehensive cost optimization report following the Microsoft Well-Architected Framework (Cost Optimization pillar) and FinOps Foundation best practices.

This skill is built to be a **single source of truth** for a resource group: it tells the *complete* cost story and is engineered to **never silently skip a resource**. A guaranteed-completeness engine (enumerate → coverage-matrix → reconcile) cross-checks **every ARM-tracked resource type** in the resource group and routes anything unmatched to a generic fallback, so no resource type is silently dropped. (Costs with no RG-scoped ARM object — inter-region/egress bandwidth, subscription-scoped Marketplace SaaS, classic ASM resources — are surfaced separately via Cost Management actuals, not the type reconcile.)

**Arguments:**
- `$0` - Resource Group name (required)
- `$1` - Region for pricing context (optional, defaults to auto-detect from RG)

**Output:** `~/azure-cost-analysis/{rg-name}-cost-analysis.md`

---

## Core Principles (read first)

1. **Guaranteed completeness — no silent drops.** Every distinct ARM type in the RG is either analyzed by a specialized agent (via [`references/resource-coverage-matrix.md`](./references/resource-coverage-matrix.md)) or by the generic fallback. The run is only COMPLETE when it can attest `N of N resource types analyzed, 0 gaps`.
2. **Cost is never null (Cost Resolution Chain).** Every resource MUST end with a cost figure **and a stated basis** — never blank, never an unexplained zero-dollar figure. Resolve in order: **(1) Billed** — Cost Management amortized cost, *only when it returns a real non-zero amount*; **(2) List-price (Retail API)** — when billed is zero/unavailable, compute `usage quantity × current unit rate` from the Azure Retail Prices API (region-correct, priced as of the run date); **(3) List-price (web)** — when the Retail Prices API has no match (Marketplace/third-party/preview/SKU gaps), WebSearch today's price, compute, and cite URL + date; **(4) Manual** — only if no price exists anywhere, flag `price unavailable — manual review`. **Credit/sponsorship rule:** on credit/sponsorship/free-trial/MSDN subscriptions Cost Management commonly reports zero because credits cover spend — a **zero billed amount does NOT mean free**, so the **list-price model becomes the authoritative headline cost** (what the workload would cost at PAYG rates / draws down from credits). [`references/pricing-reference.md`](./references/pricing-reference.md) is an offline fallback to the live Retail Prices API.
3. **Rate optimization is first-class.** Reserved Instances, Savings Plans, Azure Hybrid Benefit, and Spot are often the single largest savings lever — always analyze both "should buy" and "bought-but-wasting" signals (see [`references/commitment-discounts.md`](./references/commitment-discounts.md)).
4. **Safety-first.** Never recommend a destructive change without the protocol in [`references/safety-checklist.md`](./references/safety-checklist.md); distinguish reversible rate changes from destructive deletes.
5. **Proof-gated actions — default to INVESTIGATE.** No destructive action (DELETE / downgrade / redundancy- or retention-reduction) may appear in the report as actionable unless the **verification gate** ([`scripts/verify-findings.sh`](./scripts/verify-findings.sh), Phase 6) has independently re-proven it safe against live Azure. Anything not `VERIFIED_SAFE` is auto-demoted to **INVESTIGATE** with its evidence. The worst case the user can act on is "investigate" — never a wrong delete. (PE/connectivity deletes cap at NEEDS_CONFIRMATION by design — external references can't be proven absent.)
6. **Adaptive & cross-referenced.** Branch on the resource types actually discovered; map dependencies (App Service → Redis, Web App → Storage, PE → Private DNS) before recommending removal.
7. **Actionable.** Every finding ships the exact `az` command to remediate it, with projected savings and a risk level.

---

## Workflow

### Phase 1: Pre-flight Validation

Run the pre-flight check script to validate the environment and capabilities:

```bash
bash scripts/az-preflight-check.sh "$0"
```

The script validates Azure CLI, login, subscription, RG existence, the `resource-graph` extension, and the reachability of Cost Management (ActualCost + AmortizedCost), Azure Retail Prices API, Advisor, and the reservation/savings-plan recommendation APIs. It also emits the resolved region, subscription id, currency/cloud, the **distinct resource-type count**, and writes the authoritative discovered-type set to `~/azure-cost-analysis/$0-discovered-types.txt` and the kind-discriminated set to `~/azure-cost-analysis/$0-discriminated-kinds.txt`. The script always emits one JSON line (an `EXIT` trap guarantees output even on an unexpected abort).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Attri-Inc/azure-cost-analyzer](https://github.com/Attri-Inc/azure-cost-analyzer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
