---
trigger: always_on
description: This source file is part of the Heartwood open-source project
---

<!--

This source file is part of the Heartwood open-source project

SPDX-FileCopyrightText: 2026 Stanford University and the project authors (see CONTRIBUTORS.md)

SPDX-License-Identifier: MIT

-->

# AGENTS Instructions

Guidance for contributors working in this repository.

## Purpose

This file is the repository orientation and rule set. It should point to the canonical docs instead of restating them.

When project direction changes, update the relevant architecture or operations page first, then update this file only if routing or durable working rules change.

## Canonical Documentation

| Need | Source |
|---|---|
| Repository summary | [README.md](README.md) |
| Published documentation home and user journey | [documentation/index.md](documentation/index.md) |
| First-use installation, project, model, and interface flow | [documentation/start/index.md](documentation/start/index.md) |
| Workstation, Terra, Carina, and managed-environment selection | [documentation/platforms/index.md](documentation/platforms/index.md) |
| Installation routes | [documentation/start/install.md](documentation/start/install.md) |
| Project boundary, persistence, and `.heartwood/` layout | [documentation/start/project.md](documentation/start/project.md) |
| Research-environment, hosted, compatible-service, and Heartwood-managed model workflows | [documentation/models/index.md](documentation/models/index.md) |
| Deployment responsibilities and platform extension contract | [documentation/operate/index.md](documentation/operate/index.md) |
| Release support, compatibility, and deprecation policy | [documentation/operate/support.md](documentation/operate/support.md) |
| Browser workflow | [documentation/use/browser.md](documentation/use/browser.md) |
| Research specialist roles and boundaries | [documentation/use/specialists.md](documentation/use/specialists.md) |
| Research workflow setup, stage checks, and review | [documentation/use/research-workflows.md](documentation/use/research-workflows.md) |
| Command reference | [documentation/reference/cli.md](documentation/reference/cli.md) |
| Readiness states, stable diagnostics, and recovery steps | [documentation/reference/troubleshooting.md](documentation/reference/troubleshooting.md) |
| Qualified GPU runtime, model, and platform combinations | [documentation/reference/gpu-compatibility.md](documentation/reference/gpu-compatibility.md) |
| Product boundaries and durable technical rationale | [documentation/architecture/index.md](documentation/architecture/index.md) |
| Project, gateway, adapter, interface, and data-flow architecture | [documentation/architecture/system.md](documentation/architecture/system.md) |
| Security and controlled-data responsibilities | [documentation/operate/security.md](documentation/operate/security.md) |
| Authoritative audit exports, signing, and retention | [documentation/operate/audit-checkpoints.md](documentation/operate/audit-checkpoints.md) |
| Audit integrity and session persistence | [documentation/architecture/sessions-audit.md](documentation/architecture/sessions-audit.md) |
| Scientific experiment records, lineage, and recovery | [documentation/architecture/experiments.md](documentation/architecture/experiments.md) |
| Skill trust, distribution, activation, and interface contract | [documentation/architecture/skills.md](documentation/architecture/skills.md) |
| Research Skill contribution, policy, and validation | [documentation/contribute/skills.md](documentation/contribute/skills.md) |
| Testing layers and evidence language | [documentation/architecture/testing.md](documentation/architecture/testing.md) |
| Python and web development workflow | [documentation/contribute/development.md](documentation/contribute/development.md) |
| Pull request description structure | [Organization pull request template](https://github.com/SchmiedmayerLab/.github/blob/main/.github/pull_request_template.md) |
| Planned implementation, acceptance criteria, and delivery status | [GitHub Issues](https://github.com/SchmiedmayerLab/heartwood/issues) and the [Heartwood Project](https://github.com/orgs/SchmiedmayerLab/projects/2) |
| Acronyms and specialized terms | [documentation/reference/glossary.md](documentation/reference/glossary.md) |

## Engineering Invariants

- The process current directory is the project boundary, and project-private configuration, models, sessions, and audit state live under `.heartwood/`.
- The gateway owns project behavior, persisted settings, session mutation, and interface projections. The terminal, browser, and notebook bridge adapt the same typed contracts rather than maintaining separate business rules.
- OpenHands owns the agent loop, conversation behavior, task tracking, and coding tools. Extend its public contracts through the existing adapter instead of introducing a parallel agent or tool implementation.
- Platform-specific behavior belongs in capability, policy, detector, launcher, and packaging adapters. It must not fork the application workflow.
- Generic and platform-derived artifacts share manifests, installers, image stages, runtime locks, and qualification scripts. Parameterize real platform differences rather than copying assembly logic.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SchmiedmayerLab/heartwood](https://github.com/SchmiedmayerLab/heartwood) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
