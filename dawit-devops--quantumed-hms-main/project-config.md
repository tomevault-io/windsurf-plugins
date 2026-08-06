---
trigger: always_on
description: > **Canonical master documents** (load before any work):
---

# Agent Guidance for QuantuMed HMS

> **Canonical master documents** (load before any work):
>
> - [`docs/agents/WORKSPACE_RULES.md`](docs/agents/WORKSPACE_RULES.md) — **Part 1:** canonical stack, monorepo layout, tenancy, security, RBAC, API contract, frontend routes, i18n, data model, adapter matrix, scope boundary.
> - [`docs/agents/AGENTIC_WORKFLOW.md`](docs/agents/AGENTIC_WORKFLOW.md) — **Part 2:** agent roster, cross-agent protocols, CI/CD pipeline overview, phased (A–E) task checklists, acceptance criteria, timeline.
> - [`docs/agents/Agents.md`](docs/agents/Agents.md) — **Narrative supplement:** system context, deployment & multi-hospital plan, non-functional requirements, deliverables per sprint.
>
> **Single source of truth for the spec:** [`docs/action-plan.md`](docs/action-plan.md) — do not mutate via PRs.
> **ADRs** (committed architecture decisions): [`docs/architecture/adr/`](docs/architecture/adr/)

---

## 1. Project Goal

**QuantuMed Hospital Management System (QuantuMed HMS)** is a robust, multi-tenant web SaaS platform that supports single and multiple hospitals (multi-hospital installations), telemedicine/teleradiology, multi-language UI, full financials, pharmacy and inventory, role-based access, SMS/email automation, POS and invoicing, and third-party integrations (Stripe, local mobile-money, SMS gateways, Jitsi). The product must be production-ready, secure (HIPAA/GDPR-aware where applicable), maintainable, observable, interoperable, and internationalized (Arabic, Amharic, Oromo, Somali, Tigrigna, English).

It consolidates the **operational, clinical, financial, and collaborative** needs of hospitals, clinics, diagnostic centers, and nursing homes into a single unified SaaS platform. AI/ML capabilities are embedded from day one via an adapter boundary, not bolted on later:

- **Rule-based patient triage** (MVP) — deterministic scoring using vitals and presenting complaints.
- **Clinical Decision Support (CDS)** (limited MVP surface) — drug-interaction, allergy, dose-range, and duplicate-therapy advisories surfaced at order entry.
- **Automated Clinical QI/QA** (limited scope, MVP→Beta) — structured quality measure extraction from encounters (e.g. diabetes HbA1c adherence, hypertension control), with explainable rule attribution only — no black-box models in MVP.

### Deployment Targets

| Mode                          | Tooling                            | Use Case                  |
| ----------------------------- | ---------------------------------- | ------------------------- |
| Single-hospital quick install | `infra/docker/docker-compose.yml`  | Local / single facility   |
| Production multi-hospital     | Terraform + Kubernetes Helm charts | Multi-facility production |

Stateless app servers behind a load balancer; state in managed PostgreSQL and object storage. Availability target: < 3% downtime during business hours for MVP. Observability: OpenTelemetry traces, Prometheus metrics, structured Pino JSON logs, health endpoints (`/health/live`, `/health/ready`).

---

## 2. Source of Truth

- [`docs/action-plan.md`](docs/action-plan.md) is the canonical spec. Do not deviate without explicit user approval — propose via PR, do not merge unilaterally. **The action plan is never mutated by PRs.**
- ADRs in [`docs/architecture/adr/`](docs/architecture/adr/) record committed architectural decisions. New decisions get a new ADR; accepted ADRs are **never edited in place**.
- `docs/agents/WORKSPACE_RULES.md` is the authoritative coding rule set derived from the action plan.
- `docs/agents/AGENTIC_WORKFLOW.md` is the authoritative phased workflow and agent task reference.
- When this `AGENTS.md` extends scope beyond the action plan (mobile-money adapters, local SMS providers, extended roles, dynamic domain experts), those extensions are recorded in a tracking ADR and reflected back into Part 1 / Part 2 — the three docs are kept in sync (see §15 Doc Hygiene).

---

## 3. Multi-Agent Orchestration

A core 12-agent team collaborates under the **Orchestrator**, augmented by **dynamic domain-expert agents** spun up on demand (see §3.2).

### 3.1 Core Agent Roster

| #  | Agent | Owns |
| -- | ----- | ---- |
| 1  | **Orchestrator** (lead) | Roadmap, phase gates, release cadence, sprint backlogs, final integration checks, deploy checklists. **Only agent that approves phase promotion.** |
| 2  | **Solution Architect** | All ADRs, data partitioning/backup/scaling, infra topology (Terraform/Helm/K8s), tenancy + encryption key lifecycle. First deliverables: ADR-0001/0002/0003, approved before Phase A work begins. |
| 3  | **Backend Engineer** | All NestJS bounded-context modules, REST endpoints, DTO validation, JWT+refresh+RBAC+MFA, OpenAPI 3.1 spec, backend unit tests. |
| 4  | **Frontend Engineer** | All Next.js role dashboards, appointment calendar popup, prescription combobox, language switcher + RTL, per-tenant branding, PWA, WebSocket real-time, frontend unit tests. |
| 5  | **Database Engineer** | Prisma multi-schema definition, all migrations + indexes, field-level encryption integration, retention rules, catalog/permission/demo seeds. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dawit-devops/quantumed-hms-main-](https://github.com/dawit-devops/quantumed-hms-main-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
