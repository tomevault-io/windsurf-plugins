---
trigger: always_on
description: > **THIS FILE IS READ FIRST. BEFORE TOUCHING ANY CODE, ANY FILE, ANY DECISION.**
---

# CLAUDE.md — AutoStack AI Operating Manual

> **THIS FILE IS READ FIRST. BEFORE TOUCHING ANY CODE, ANY FILE, ANY DECISION.**
> If you are an AI agent working in this repository, read this entire file before doing anything else.

---

## What AutoStack Is

AutoStack is a **production-grade, unified deployment and management platform**.

It allows developers and platform teams to:
- Deploy containerized applications to **Kubernetes clusters** (already working, do not break)
- Deploy containerized applications to **cloud providers** (AWS ECS Fargate, Google Cloud Run, Azure ACA) — this is being built
- Manage deployments with **real-time observability** (logs, metrics, events via WebSocket)
- Automate updates through **image polling and rollout policies**
- Control infrastructure with **blueprint templates** and versioned rollout history
- Estimate and track **cloud costs** before and after deployment
- Operate with **AI-assisted incident explanation** and right-sizing recommendations

The Kubernetes deployment system is **complete and production-ready**. Cloud provider support is **in active development**.

---

## The Single Most Important Rule

```
THE KUBERNETES SYSTEM WORKS.
DO NOT BREAK IT.
DO NOT REFACTOR IT WITHOUT EXPLICIT INSTRUCTION.
DO NOT CHANGE THE CRD SCHEMA WITHOUT EXPLICIT INSTRUCTION.
DO NOT TOUCH THE OPERATOR UNLESS SPECIFICALLY ASKED.
```

Everything cloud-related is **additive**. New files. New collections. New services. Nothing replaces existing Kubernetes functionality. If you are ever unsure whether a change touches the Kubernetes path, **stop and ask**.

See `KUBERNETES_EXISTING_SYSTEM.md` for the full map of what must never be disturbed.

---

## Repository Identity

- **Backend**: Go — Kubernetes operator pattern, PocketBase server, WebSocket hub, background reconciliation service
- **Frontend**: SvelteKit — real-time dashboard, deployment controls, log streaming
- **Database**: PocketBase (SQLite-backed, migrating to PostgreSQL path available but not yet executed)
- **Orchestration**: Custom Kubernetes CRD `one-click.dev/v1alpha1 Rollout` — the operator watches this
- **Auth**: PocketBase built-in auth (OAuth2 social login) — SSO/SAML is planned, not yet built
- **Primary CRD Group**: `one-click.dev/v1alpha1`
- **Operator Namespace**: `autostack` (assumed — confirm before changing)
- **Frontend Port**: varies by environment (see ENVIRONMENT_MATRIX.md)
- **Backend Port**: varies by environment (see ENVIRONMENT_MATRIX.md)

---

## Critical Priorities (In Order)

1. **Working Kubernetes system stays working.** This is non-negotiable.
2. **PocketBase is the single source of truth** for all deployment desired state — both Kubernetes and cloud.
3. **Security is never an afterthought.** Credentials are never logged. Secrets are never stored in plaintext. See `SECURITY_AND_ACCESS.md`.
4. **Cloud deployments are additive.** New provider implementations live behind the Provider interface. No cloud-specific `if provider == "aws"` logic in core paths.
5. **Cost estimates are honest ranges, never promises.** Never hardcode pricing. Always call pricing APIs. Always show uncertainty.

---

## AI Behavioral Rules

### Before You Touch Anything
- Read the relevant section of `ARCHITECTURE.md` for the area you are working in
- Read `SYSTEM_BOUNDARIES.md` to understand what layer owns what
- Read `DATA_MODEL.md` if you are changing any PocketBase collection or schema
- Read `KNOWN_ISSUES.md` so you don't fix things that are intentionally deferred
- Read `DECISIONS.md` so you don't reintroduce patterns that were deliberately rejected

### While Working
- Make **incremental, targeted changes**. Never rewrite a file unless explicitly asked.
- If you are modifying a file, understand every function in it first.
- If you add a dependency, add it to `DECISIONS.md` with rationale.
- If you make an architectural decision (even small), document it.
- Never guess at schema fields. Look at the actual PocketBase collection definitions.
- Never assume a port, URL, or environment variable. Check `ENVIRONMENT_MATRIX.md`.

### When You Are Uncertain
- **Stop and ask.** Uncertainty in infrastructure code causes outages.
- State what you know, what you don't know, and what you need clarified.
- Never silently make an assumption about security, credentials, or destructive operations.

### Things You Must Never Do Without Explicit Human Approval
- Delete or modify PocketBase collections that the Kubernetes system uses
- Change the CRD schema (`one-click.dev/v1alpha1`)
- Modify the Kubernetes operator reconciliation loop
- Change authentication logic
- Add or remove encrypted fields in credential storage
- Modify rollout history or audit log logic
- Execute any infrastructure destroy operation
- Change RBAC permissions on the operator

---

## Repository Conventions

### Folder Structure (Top-Level)
```
/                           → root
/cmd/                       → Go entrypoints (operator, server, reconciler)
/pkg/                       → Go packages
  /pkg/providers/           → Cloud provider implementations (Provider interface)
  /pkg/kubernetes/          → Kubernetes operator logic (DO NOT TOUCH without instruction)
  /pkg/reconciler/          → Cloud reconciliation service

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Raj-glitch-max/AutoStack-GO-Svelte](https://github.com/Raj-glitch-max/AutoStack-GO-Svelte) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
