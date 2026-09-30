---
trigger: always_on
description: This repository provides the foundational hub and managed-cluster components for
---

# multicloud-operators-foundation Agent Instructions

This repository provides the foundational hub and managed-cluster components for
Red Hat Advanced Cluster Management (ACM). It is a Go/Kubernetes codebase using
controller-runtime, client-go, and Open Cluster Management APIs.

## Repository layout

- `cmd/controller/`: hub-side foundation controller and controller-runtime manager.
- `cmd/agent/`: managed-cluster work-manager agent and its controllers.
- `pkg/controllers/`: hub-side reconcilers for cluster information, cluster sets,
  RBAC, image registries, add-ons, garbage collection, and managed service accounts.
- `pkg/klusterlet/`: agent-side action, view, cluster information, and node collection
  controllers.
- `pkg/webhook/`: admission webhook implementations.
- `hack/`: CRD, code-generation, and protobuf update/verification scripts.
- `deploy/`, `examples/`, and `docs/`: deployment manifests, examples, and component
  documentation.
- `test/`: unit, envtest integration, end-to-end, and performance tests.

For system architecture, data flows, and module boundaries, see
[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Development commands

Run commands from the repository root:

```bash
make build
go test ./pkg/...
make test-integration
make verify
```

- `make build` builds the Go binaries through the OpenShift build-machinery include.
- `go test ./pkg/...` runs the package unit tests without requiring a cluster.
- `make test-integration` provisions envtest assets, compiles `test/integration`, and
  runs the integration suite.
- `make test-e2e` requires configured Hub and managed clusters and deployment assets.
- `make verify` checks generated CRDs and generated code; use `make update` only when
  intentionally regenerating them.
- `make images` builds the `quay.io/stolostron/multicloud-manager:latest` image.

Changes to generated CRDs or code must be made through the corresponding source and
generation scripts, then verified with `make verify`. Keep controller behavior,
RBAC, health probes, and leader-election semantics covered by tests when changing
reconciliation logic.

## Tool availability

- GitHub operations: GitHub MCP tools are available; the `gh` CLI is not assumed.
- Jira operations: Jira MCP tools are available; the `jira` CLI is not assumed.
- If multiple GitHub organization tokens are configured, use the `GH_TOKEN_<ORG>`
  convention without printing token values.

## Personal configuration

Read personal config at the start of any task that needs an assignee, email, or
project key. Canonical path: `~/.config/user.local.md` (tool-agnostic, global).
If the file does not exist, fall back to agent memory (`user-config`), then
placeholders. Run `make personalize` to generate or update the file when Fleet
Engineering tooling is available; this repository does not currently define that
target.

## Fleet Engineering Skills

Fetch and apply the relevant skill when the task matches its domain. The canonical
catalog is [Fleet Engineering skills](https://github.com/OpenShift-Fleet/agentic-sdlc/blob/main/skills/README.md).

| Skill | When to use |
|---|---|
| [bug-specialist](https://raw.githubusercontent.com/OpenShift-Fleet/agentic-sdlc/main/skills/bug-specialist/SKILL.md) | Bug triage, reproduction, and fix planning |
| [epic-specialist](https://raw.githubusercontent.com/OpenShift-Fleet/agentic-sdlc/main/skills/epic-specialist/SKILL.md) | Multi-sprint epics with outcomes |
| [feature-specialist](https://raw.githubusercontent.com/OpenShift-Fleet/agentic-sdlc/main/skills/feature-specialist/SKILL.md) | Large customer-facing capabilities |
| [initiative-specialist](https://raw.githubusercontent.com/OpenShift-Fleet/agentic-sdlc/main/skills/initiative-specialist/SKILL.md) | Multi-team strategic programs |
| [jira-create](https://raw.githubusercontent.com/OpenShift-Fleet/agentic-sdlc/main/skills/jira-create/SKILL.md) | Interactive Jira issue creation |
| [jira-qe-readiness](https://raw.githubusercontent.com/OpenShift-Fleet/agentic-sdlc/main/skills/jira-qe-readiness/SKILL.md) | Check whether a Jira ticket is ready for QE |
| [jira-report](https://raw.githubusercontent.com/OpenShift-Fleet/agentic-sdlc/main/skills/jira-report/SKILL.md) | Produce Jira portfolio and quality reports |
| [jira-specialist](https://raw.githubusercontent.com/OpenShift-Fleet/agentic-sdlc/main/skills/jira-specialist/SKILL.md) | General Jira triage and issue management |
| [jira-type-audit](https://raw.githubusercontent.com/OpenShift-Fleet/agentic-sdlc/main/skills/jira-type-audit/SKILL.md) | Audit Jira issue types across a hierarchy |
| [outcome-specialist](https://raw.githubusercontent.com/OpenShift-Fleet/agentic-sdlc/main/skills/outcome-specialist/SKILL.md) | Strategic outcomes tied to OKRs |
| [release-dod](https://raw.githubusercontent.com/OpenShift-Fleet/agentic-sdlc/main/skills/release-dod/SKILL.md) | Build a release Definition of Done checklist |
| [risk-report](https://raw.githubusercontent.com/OpenShift-Fleet/agentic-sdlc/main/skills/risk-report/SKILL.md) | Detect risk signals and draft a status report |
| [risk-specialist](https://raw.githubusercontent.com/OpenShift-Fleet/agentic-sdlc/main/skills/risk-specialist/SKILL.md) | Manage risk registers and mitigations |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stolostron/multicloud-operators-foundation](https://github.com/stolostron/multicloud-operators-foundation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
