---
trigger: always_on
description: This is the working guide for anyone changing OpenLakeForge — human or coding
---

# Agent and contributor guide

This is the working guide for anyone changing OpenLakeForge — human or coding
agent. It records what the repository actually enforces today. If a statement
here disagrees with the code, the code wins and this file is a bug.

`CLAUDE.md` points here. There is one copy of these rules on purpose.

## What this project is

OpenLakeForge is a cloud-agnostic, self-hostable lakehouse platform: open-source
components assembled on Kubernetes with Terraform and Helm. The data path is
CSV → Bronze → Floe validation → Silver Iceberg through Polaris → dbt-trino Gold
→ Trino → Superset, orchestrated by Dagster.

The audience is small data teams self-hosting a complete lakehouse. Runtime
footprint, onboarding friction, and recoverability are product features here,
not polish — a team without a platform engineer cannot absorb a stack that needs
one. `v0.2-alpha` is scoped around exactly that; see the roadmap.

## Non-goals

These are settled scope decisions, not gaps. A change that adds machinery for
one of them is out of scope, and a review finding that asks for it should be
closed with a pointer here rather than fixed.

| Not a goal | Why |
| --- | --- |
| Concurrent or re-entrant `olf` invocation | `olf` is a single-operator CLI: one invocation at a time, on a workstation or a CI runner. `olf deploy` runs its phases sequentially in one process and relies on that. Do not add locks, leases, atomic-rename dances, or TOCTOU guards to shared code. |
| Windows support | Deployment targets are Linux and macOS. |

The audience is a small data team without a platform engineer, and that is a
design constraint rather than a market description: machinery those teams will
never exercise is not free. It costs runtime footprint, onboarding friction,
and reviewer attention — the three things this project treats as product
features.

This list is about the CLI process model, not about the deployed platform's
security. Authentication, credential scope, and stage data isolation in the
running stack are real requirements — open items are tracked as debt in
`docs/technical-debt.md`, not waived here.

## Orientation — read in this order

1. `README.md` — stack, deployment targets, local workflow
2. `docs/industrialization-roadmap.md` — milestones, release gates, what is
   delivered and what is not
3. `docs/architecture/overview.md` and `docs/architecture/provider-contracts.md`
4. `docs/adr/README.md` — the decision log index; each ADR describes what
   binds today, all worth reading
5. `docs/technical-debt.md` — the live debt register with a fix path per item
6. `docs/architecture/diagrams/README.md` — pod census and runtime topology

Current work is tracked in GitHub milestones. Issues carry `priority: P0/P1/P2`
labels expressing intended sequence within a milestone.

## Repository map

| Path | Contains |
| --- | --- |
| `lakehouse_code/bronze/<source>/` | Source-owned: `source.yaml` descriptor plus dlt extract |
| `lakehouse_code/silver/<domain>/` | Domain-owned: Floe contracts and transformations |
| `lakehouse_code/gold/<product>/` | Product-owned: dbt project |
| `lakehouse_code/dashboards/superset/<dashboard>/` | Consumption-owned: Superset reports |
| `lakehouse_code/pipelines/dagster/` | User-maintained Dagster orchestration code |
| `lakehouse_code/lakehouse.yaml` | Canonical domain/product business metadata descriptor |
| `openlakeforge.yaml` | Project-root Deployment Profile v1; parsed and resolved by `olf profile validate`/`resolve` (ADR 0011) |
| `openlakeforge.conformance.yaml` | Local DEV+PROD conformance profile (#155); the nightly applies it verbatim (#220) |
| `libs/` | Shared runtime Python imported by the project-code image |
| `packages/domain-model/` | Canonical provider-neutral descriptor and inventory package |
| `tools/olf/` | The `olf` CLI — uv-managed deploy tooling, contracts, artifacts, scaffolding, e2e |
| `infra/terraform/environments/` | Per-environment wiring; `contracts.tf` is the contract surface |
| `infra/terraform/foundations/` | Cluster and registry creation (kind, AKS, EKS) |
| `infra/terraform/modules/` | Component modules grouped by capability |
| `infra/helm/values/local/` | Helm values for the local profile |
| `images/project-code/` | The Dagster runtime image |
| `release/component-catalog.yaml` | Immutable version, digest pins, and the distribution version contract both `pyproject.toml` files must match |
| `docs/schema/lakehouse.schema.json`, `docs/schema/source.schema.json` | Schemas `lakehouse.yaml` and `source.yaml` validate against |
| `docs/schema/deployment-profile.schema.json` | Schema for Deployment Profiles |

## Architectural rules

These are load-bearing. Breaking one means the change is wrong even if it works.

1. **Contracts before providers.** Components communicate through the provider
   contracts in `contracts.tf`, never directly with a provider. Adding a
   capability means extending the contract, then writing an adapter.
2. **Descriptors stay provider-neutral.** `lakehouse_code/lakehouse.yaml` and
   `lakehouse_code/bronze/<source>/source.yaml` must not name Polaris, Glue, S3,
   or SeaweedFS. Physical catalog and object-store names are derived from these

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenLakeForge/openlakeforge](https://github.com/OpenLakeForge/openlakeforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
