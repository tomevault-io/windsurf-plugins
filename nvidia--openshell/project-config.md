---
trigger: always_on
description: This file is the primary instruction surface for agents contributing to OpenShell. It is injected into your context on every interaction — keep that in mind when proposing changes to it.
---

# Agent Instructions

This file is the primary instruction surface for agents contributing to OpenShell. It is injected into your context on every interaction — keep that in mind when proposing changes to it.

See [CONTRIBUTING.md](CONTRIBUTING.md) for build instructions, task reference, project structure, and the full agent skills table.

## Project Identity

OpenShell is built agent-first. We design systems and use agents to implement them — this is not vibe coding. The product provides safe, sandboxed runtimes for autonomous AI agents, and the project itself is built using the same agent-driven workflows it enables.

## Skills

Agent skills live in `.agents/skills/`. Your harness can discover and load them natively — do not rely on this file for a full inventory. The detailed skills table is in [CONTRIBUTING.md](CONTRIBUTING.md) (for humans).

## Workflow Chains

These pipelines connect skills into end-to-end workflows. Individual skill files don't describe these relationships.

- **Community inflow:** `triage-issue` → `create-spike` → `build-from-issue`
  - Triage assesses and classifies community-filed issues. Spike investigates unknowns. Build implements.
- **Internal development:** `create-spike` → `build-from-issue`
  - Spike explores feasibility, then build executes once `state:agent-ready` is applied by a human.
- **Security:** `review-security-issue` → `fix-security-issue`
  - Review produces a severity assessment and remediation plan. Fix implements it. Both require the `topic:security` label; fix also requires `state:agent-ready`.
- **Policy iteration:** `openshell-cli` → `generate-sandbox-policy`
  - CLI manages the sandbox lifecycle; policy generation authors the YAML constraints.

## Architecture Overview

| Path | Components | Purpose |
|------|-----------|---------|
| `crates/openshell-cli/` | CLI binary | User-facing command-line interface |
| `crates/openshell-server/` | Gateway server | Control-plane API, sandbox lifecycle, auth boundary |
| `crates/openshell-sandbox/` | Sandbox runtime | Container supervision, policy-enforced egress routing |
| `crates/openshell-policy/` | Policy engine | Filesystem, network, process, and inference constraints |
| `crates/openshell-router/` | Privacy router | Privacy-aware LLM routing |
| `crates/openshell-bootstrap/` | Cluster bootstrap | K3s cluster setup, image loading, mTLS PKI |
| `crates/openshell-ocsf/` | OCSF logging | OCSF v1.7.0 event types, builders, shorthand/JSONL formatters, tracing layers |
| `crates/openshell-core/` | Shared core | Common types, configuration, error handling |
| `crates/openshell-providers/` | Provider management | Credential provider backends |
| `crates/openshell-tui/` | Terminal UI | Ratatui-based dashboard for monitoring |
| `crates/openshell-vm/` | MicroVM runtime | Experimental, work-in-progress libkrun-based VM execution |
| `crates/openshell-driver-kubernetes/` | Kubernetes compute driver | In-process `ComputeDriver` backend for K8s sandbox pods |
| `crates/openshell-driver-vm/` | VM compute driver | Standalone libkrun-backed `ComputeDriver` subprocess (embeds its own rootfs + runtime) |
| `python/openshell/` | Python SDK | Python bindings and CLI packaging |
| `proto/` | Protobuf definitions | gRPC service contracts |
| `deploy/` | Docker, Helm, K8s | Dockerfiles, Helm chart, manifests |
| `docs/` | Published docs | MDX pages, navigation, and content assets |
| `fern/` | Docs site config | Fern site config, components, and theme assets |
| `.agents/skills/` | Agent skills | Workflow automation for development |
| `.agents/agents/` | Agent personas | Sub-agent definitions (e.g., reviewer, doc writer) |
| `architecture/` | Architecture docs | Design decisions and component documentation |

## Vouch System

- First-time external contributors must be vouched before their PRs are accepted. The `vouch-check` workflow auto-closes PRs from unvouched users.
- Org members and collaborators bypass the vouch gate automatically.
- Maintainers vouch users by commenting `/vouch` on a Vouch Request discussion. The `vouch-command` workflow appends the username to `.github/VOUCHED.td`.
- Skills that create PRs (`create-github-pr`, `build-from-issue`) should note this requirement when operating on behalf of external contributors.

## Issue and PR Conventions

- **Bug reports** must include an agent diagnostic section — proof that the reporter's agent investigated the issue before filing. See the issue template.
- **Feature requests** must include a design proposal, not just a "please build this" request. See the issue template.
- **PRs** must follow the PR template structure: Summary, Related Issue, Changes, Testing, Checklist.
- **PRs from unvouched external contributors** are automatically closed. See the Vouch System section above.
- **Security vulnerabilities** must NOT be filed as GitHub issues. Follow [SECURITY.md](SECURITY.md).
- Skills that create issues or PRs (`create-github-issue`, `create-github-pr`, `build-from-issue`) should produce output conforming to these templates.

## Plans


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA/OpenShell](https://github.com/NVIDIA/OpenShell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
