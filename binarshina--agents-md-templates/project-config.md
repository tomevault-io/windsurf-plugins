---
trigger: always_on
description: > Template for AI coding agents. Replace every placeholder in `<angle_brackets>` with real project data before adopting this file. Remove example rows you do not need, but prefer filling sections over deleting them. This file should be concrete, current, and operational.
---

# AGENTS.md

> Template for AI coding agents. Replace every placeholder in `<angle_brackets>` with real project data before adopting this file. Remove example rows you do not need, but prefer filling sections over deleting them. This file should be concrete, current, and operational.

---

## Purpose

This file is the primary operating manual for AI agents working in this repository.

It should give the agent enough context to:

- understand what the project does;
- follow the correct architecture and conventions;
- place code in the right directories;
- run the right commands for setup, validation, and testing;
- avoid unsafe or low-quality changes;
- know when to stop and ask a human.

If this file is vague, outdated, or contradictory, agent output will degrade. Keep it specific.

---

## Project Snapshot

- Project name: `<project_name>`
- Project type: `<web app | api | mobile app | cli | library | monorepo | service | internal tool | other>`
- One-line description: `<what this project does in one sentence>`
- Primary users: `<who uses this system>`
- Business/domain context: `<domain, workflow, or problem space>`
- Lifecycle stage: `<prototype | mvp | production | mature | legacy modernization>`
- Maintainers / owning team: `<team_name_or_people>`
- Default branch: `<main_branch_name>`
- Repo status notes: `<active refactor | migration in progress | legacy areas | frozen modules | none>`

### Fill-In Guidance

- Write the description for an engineer, not for marketing.
- If the system is legacy or mid-migration, say so explicitly.
- If the repository contains multiple products or apps, mention that here and define the structure below.

### Example

```md
- Project name: Acme Orders
- Project type: Monorepo
- One-line description: Internal platform for creating, pricing, and tracking wholesale orders.
- Primary users: Sales operators, finance team, warehouse integrations
- Business/domain context: B2B commerce
- Lifecycle stage: Production
- Maintainers / owning team: Orders Platform Team
- Default branch: main
- Repo status notes: Legacy pricing flow is being migrated into feature-owned modules
```

---

## Agent Principles

Unless the user explicitly asks otherwise, the agent should:

- prefer the smallest safe change that solves the task;
- preserve existing architecture and naming conventions;
- update tests when behavior changes;
- update docs, config, or examples when they become stale because of the change;
- verify work before finishing;
- avoid speculative refactors;
- ask before destructive, irreversible, expensive, or production-affecting operations.

### Optimize For

1. Correctness
2. Maintainability
3. Speed

### Never Do These By Default

- Rewrite architecture without being asked.
- Introduce a new dependency when an existing project dependency can solve the problem.
- Manually edit generated files if the intended workflow is regeneration.
- Ignore failing checks related to the files or behavior you changed.
- Guess around security-sensitive, billing-sensitive, or compliance-sensitive behavior.

---

## Sources Of Truth

Consult these references before making non-trivial changes:

| Source | Path / URL | When To Use It |
| --- | --- | --- |
| Product or domain docs | `<path_or_url>` | `<behavior rules / business logic>` |
| Architecture docs | `<path_or_url>` | `<module boundaries / system design>` |
| ADRs / decisions log | `<path_or_url>` | `<why previous tradeoffs were made>` |
| API contracts / schemas | `<path_or_url>` | `<endpoint, event, or schema changes>` |
| Design system / UI docs | `<path_or_url>` | `<UI behavior, components, tokens>` |
| Contribution guide | `<path_or_url>` | `<repo workflow, branching, review expectations>` |
| Security docs | `<path_or_url>` | `<auth, secrets, permissions, compliance>` |
| Deployment / runbooks | `<path_or_url>` | `<environment, release, infrastructure changes>` |

If documentation and code disagree, prefer `<code | docs | ask_human>` and mention the mismatch in your final summary.

### Example

```md
| Source | Path / URL | When To Use It |
| --- | --- | --- |
| Architecture docs | `docs/architecture.md` | Changes to module boundaries or shared packages |
| ADRs / decisions log | `docs/adr/` | New abstractions, workflow changes, tradeoff decisions |
| API contracts / schemas | `openapi/openapi.yaml` | Any endpoint or generated client change |
```

---

## Tech Stack

Do not write “latest”. Use exact versions or supported ranges.

### Core Stack

- Language(s): `<language_and_versions>`
- Runtime(s): `<runtime_and_versions>`
- Framework(s): `<frameworks_and_versions>`
- Package manager(s): `<package_managers_and_versions>`
- Build tool(s): `<build_tools_and_versions>`
- Database(s): `<db_and_versions>`
- Messaging / queueing: `<tool_or_none>`
- Cache / storage: `<tool_or_none>`
- Hosting / infrastructure: `<cloud | on-prem | hybrid | other>`

### Key Libraries And Services

List the important tools the agent should recognize immediately:

| Area | Library / Service | Version | Purpose | Notes / Constraints |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [binarshina/agents-md-templates](https://github.com/binarshina/agents-md-templates) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
