---
trigger: always_on
description: Authoritative context for AI agents and contributors. These rules derive from
---

# LandingZone-IaC — Copilot / Agent Instructions

Authoritative context for AI agents and contributors. These rules derive from
`.specify/memory/constitution.md` (v1.1.0). The constitution wins on any conflict.

## What this repo is
An Azure Landing Zone delivered entirely as Infrastructure as Code, grown one
capability at a time, reproducibly deployable across multiple environments and
multiple Azure subscriptions with **zero manual drift**.

## Non-negotiable guardrails (from the constitution)
- **GitOps / source of truth**: Git is the single source of truth. All changes
  flow through reviewed pull requests. NEVER change live environments directly or
  via the portal (no click-ops) in managed scope.
- **Reproducibility & parity**: Identical code across all environments/
  subscriptions. Differences are expressed ONLY as declared configuration —
  never divergent code paths or branches per environment.
- **Idempotence**: Re-running a deploy against unchanged desired state MUST
  produce no changes.
- **Incremental & reversible**: Every change is small, independently deployable,
  and has a defined rollback/roll-forward strategy.
- **Security by default**: No secrets in code — ever. Least-privilege access by
  default. All resources auditable with diagnostic settings enabled.
- **Automation-first**: Everything is buildable by humans and agents
  interchangeably; operations are deterministic and machine-readable.
- **Documentation is part of "done"**: Update module docs, decision records
  (ADRs), and runbooks in the SAME pull request as the change.

## Baseline technology decisions
- **Dual IaC engines**: Support BOTH Bicep and Terraform. Each capability exposes
  an equivalent, standardized module interface (inputs/outputs/naming/tags) across
  both engines. Engine choice is declared configuration. Parity is a testable
  requirement — keep the two paths behaviorally identical.
- **Azure Verified Modules (AVM) first**: Compose from AVM modules wherever one
  exists, via specialized single-responsibility agents. Pin module versions.
  Custom modules only where no suitable AVM module exists, and only with written
  justification.
- **CI/CD & GitOps**: GitHub Actions/Workflows. PR runs plan/preview; promotion is
  lower-to-higher environment. Authenticate to Azure with **federated OIDC
  (workload identity)** — NO long-lived credentials in the repo.
- **Private by default**: Every resource is within-VNet — private endpoints,
  private DNS, no public exposure unless an explicit, reviewed, justified
  exception is declared in configuration.
- **Containers**: AKS OR Azure Container Apps (ACA), selectable by configuration,
  both private by default.
- **API governance**: Azure API Management (APIM) is the default policy enforcement
  point, but keep it behind a stable, documented gateway interface so an
  alternative (e.g., Kong/KGateway) can be substituted without changing dependents.
- **External access**: Azure Front Door OR Application Gateway (selectable), with
  WAF enabled by default.
- **Identity & secrets**: Managed identities preferred; Microsoft Entra RBAC with
  least privilege; all secrets/certs in Azure Key Vault (referenced, never inlined).
- **Observability**: Central Log Analytics + Azure Monitor + Application Insights;
  diagnostic settings on by default for every resource.
- **Python**: Used where needed for tooling/validation/glue. Provide Python
  Notebooks for onboarding, health checks, and post-deploy verification.

## Module contract (every capability)
Each module MUST declare and document:
- **Purpose**, **inputs**, **outputs**, **dependencies**, **assumptions**,
  **usage examples**.
- Explicit interface — no hidden cross-module dependencies, no hardcoded
  environment- or subscription-specific values (pass them as inputs).
- Independently versioned, independently testable, composable without modification.

## Naming & tagging
- Apply the single standardized naming convention uniformly across Bicep and
  Terraform.
- Apply the mandatory tag set (including cost-attribution tags) to every resource.

## Validation gates — what "green" means
A change is mergeable only when ALL pass together:
1. **Static validation** (format/lint/static analysis) — blocking.
2. **Policy/compliance checks** (e.g., Azure Policy) — blocking.
3. **Plan/preview review** — the plan shows ONLY intended changes, zero
   unexplained diffs (verify for both Bicep and Terraform paths where applicable).
4. **Post-deploy verification** — asserts observable outcomes (existence,
   configuration, health), validated in a lower environment before promotion.

## Do / Don't
- DO keep changes small, reviewable, and traceable to a spec/plan/task.
- DO record architecturally significant decisions as versioned ADRs.
- DO flag unknowns as `[NEEDS CLARIFICATION]` in specs.
- DON'T introduce dead or commented-out code.
- DON'T add public endpoints, secrets, or standing broad privileges by default.
- DON'T create per-environment code divergence — use configuration.
- DON'T skip the validation gate or promote on a failed/absent gate.

## Working with Spec Kit
- Constitution: `.specify/memory/constitution.md`.
- Flow per capability: `/speckit.specify` → `/speckit.clarify` → `/speckit.plan`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pkumar26/LandingZone-IaC](https://github.com/pkumar26/LandingZone-IaC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
