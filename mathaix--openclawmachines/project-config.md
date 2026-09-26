---
trigger: always_on
description: This file gives coding agents shared project context. It is intentionally
---

# OpenClaw Machines Agent Guide

This file gives coding agents shared project context. It is intentionally
tool-neutral: use it from Claude, Codex, Gemini, or any other coding agent.

## Mission

OpenClaw Machines is the Apache-2.0 public core for running AI agents in
KVM-backed Firecracker microVMs on infrastructure the operator controls.

The public core should provide:

- A minimum control plane that can run locally or as an operator-hosted service.
- Host enrollment and worker agents for KVM hosts.
- Machine lifecycle, placement, routing primitives, and runtime telemetry.
- Firecracker/rootfs/runtime pieces needed to boot isolated OpenClaw machines.
- API surfaces that remain compatible with the separate `ocm` CLI project.

The public core is not the commercial hosted overlay. Billing, plan enforcement,
pricing, reservations, hosted-only admin, customer management, private
infrastructure runbooks, and confidential deployment assumptions belong in the
private overlay.

Rule of thumb: if a feature is required to run OpenClaw Machines yourself, keep
it public and provider-neutral. If it sells, gates, meters, or administers the
hosted business, keep it out of this repo.

## Architecture

The target production-like architecture is:

```text
User
  -> Cloudflare edge
  -> Cloudflare Tunnel
  -> OCM control plane
  -> OCM-issued ocm_token
  -> Cloudflare Worker + KV data plane
  -> KVM worker tunnel
  -> Firecracker VM
```

Local development can use trusted dev auth and localhost services. A
self-hosted/operator deployment should preserve the hosted architecture:
Cloudflare Tunnel ingress, Worker/KV data-plane routing, Firebase or Cloudflare
Access human auth, OCM-issued `ocm_token`, enrolled KVM workers, and Firecracker
VMs. Do not replace this with a different auth provider, routing layer, or direct
public exposure of worker hosts.

Human auth modes currently documented for operator deployments:

- Firebase: app login inside OCM, then `/api/auth/session/exchange`.
- Cloudflare Access: identity enforced at the edge, then resolved by OCM.

Workers are infrastructure, not human users. Authenticate workers with OCM
enrollment/agent tokens and optional Cloudflare service tokens, not Firebase.

## Repository Layout

- `backend/`: Go control plane, worker/agent APIs, provisioning, metadata,
  routing, orchestration, auth, and store code.
- `backend/cmd/server`: control-plane server.
- `backend/cmd/agent`: KVM host worker agent.
- `backend/cmd/authproxy`: guest auth proxy.
- `backend/cmd/ocm-secrets`: helper for secret material.
- `frontend/`: React + TypeScript + Vite UI.
- `worker/`: Cloudflare Worker data-plane routing code.
- `rootfs/` and `scripts/init-*.sh`: guest image/runtime initialization.
- `scripts/`: local development, preflight, image, runtime, and test helpers.
- `ci/` and `.github/workflows/`: public CI and trusted KVM integration lanes.
- `docs/`: public-core setup, operator, CI, and boundary documentation.
- `llms/`: copy-pasteable operator runbooks for LLM-driven setup.
- `.agents/`: repo-local agent skills and maintainer notes for repeatable
  review, testing, debugging, and self-hosted workflows.

The `ocm` CLI lives in the separate Apache-2.0 repository:
<https://github.com/mathaix/ocm-cli>. Do not reintroduce or patch CLI
implementation code in this repo. Keep API contracts and docs compatible with
that project.

## Read First

Before changing architecture, deployment behavior, auth, routing, host
enrollment, Firecracker runtime, or public docs, read the relevant docs:

- `README.md`: public-core overview and scope.
- `docs/overlay-boundary.md`: public core vs private overlay boundary.
- `docs/local-setup.md`: trusted local/BYO-host development path.
- `docs/control-plane-profiles.md`: `local`, `operator`, and `hosted` profile
  semantics.
- `docs/self-hosted-control-plane.md`: hosted-parity self-hosted prerequisites.
- `docs/ci-release.md`: public CI vs trusted KVM/release lanes.
- `docs/kvm-integration-ci.md`: maintainer-gated KVM runner contract.
- `docs/cli.md`: relationship to `mathaix/ocm-cli`.
- `llms/self-hosted-setup.txt`: operator setup workflow for LLM agents.

When behavior changes, update the relevant docs in the same change.

Repo-local agent skills live under `.agents/skills/`. Use them as focused
workflow overlays after reading this guide:

- `.agents/skills/ocm-testing/SKILL.md`: choosing the smallest safe proof.
- `.agents/skills/ocm-review/SKILL.md`: PR/review-comment/code-review posture.
- `.agents/skills/ocm-debugging/SKILL.md`: boundary-first debugging.
- `.agents/skills/ocm-self-hosted/SKILL.md`: local/operator/self-hosted work.

Maintainer notes under `.agents/maintainer-notes/` capture project decisions
that should be enforced during review, especially routing and runtime
invariants.

## Operating Rules

- Prefer `make` targets over invoking scripts directly. The Makefile documents
  the supported local commands.
- Keep changes tightly scoped. Avoid unrelated refactors, broad doc scrubs, or
  generated churn.
- Do not commit secrets, tokens, private URLs, customer data, hosted-only
  runbooks, or private overlay assumptions.
- Never print secret values in logs or final reports. Redact tokens, keys,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mathaix/OpenClawMachines](https://github.com/mathaix/OpenClawMachines) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
