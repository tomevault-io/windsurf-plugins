---
trigger: always_on
description: This repository is a design and verification kit for production AI-enabled systems. It serves FDEs, internal applied-AI engineers, product teams, and operations leaders. Optimize for an accepted business outcome that can be independently verified, not for agent autonomy, tool count, model novelty, or architectural complexity.
---

# The FDE Guide: Agent Working Contract

## Mission

This repository is a design and verification kit for production AI-enabled systems. It serves FDEs, internal applied-AI engineers, product teams, and operations leaders. Optimize for an accepted business outcome that can be independently verified, not for agent autonomy, tool count, model novelty, or architectural complexity.

Treat an agent as one component option. For each consequential decision, first compare deterministic code, optimization, classical ML, retrieval, a foundation-model call, a bounded agent workflow, and human review as applicable. Select the smallest sufficient mechanism and preserve the authority, evidence, cost, fallback, and retirement rationale. `ARC-004`, `ARC-005`.

Use [`README.md`](README.md) for the human-facing introduction. Use this file as the working contract for repository navigation and changes.

## Required orientation

Before changing a technical artifact:

1. Inspect [`catalog.json`](catalog.json) to resolve governed artifact IDs and paths.
2. Select the applicable task route below.
3. Follow that route's order; load controls, schemas, blueprints, examples, and evidence only when applicable.
4. Read [`README.md`](README.md) when changing public positioning or navigation.
5. Expand context only when the inspected artifact reveals another dependency.

Do not load the entire repository by default. Use the task routes below, then follow only the direct links needed to complete the work.

## Repository map

| Path | Role | Treat it as |
| --- | --- | --- |
| [`catalog.json`](catalog.json) | Lists governed artifacts, types, paths, and tags | Registry; update when a cataloged artifact is added, moved, or removed |
| [`controls/`](controls/control-catalog.json) | Defines production requirements and release gates | Engineering policy normative within this guide |
| [`schemas/`](schemas/README.md) | Defines valid structures for machine-readable artifacts | Structural source of truth |
| [`playbooks/`](playbooks/README.md) | Connects field discovery, value, delivery, adoption, handoff, and operation | End-to-end FDE lifecycle |
| [`blueprints/`](blueprints/README.md) | Defines reference components, boundaries, states, failures, and release tests | Architecture starting points, not mandatory frameworks |
| [`templates/`](templates/README.md) | Provides starter design artifacts | Starting material that must be adapted and completed for the target workflow |
| [`examples/`](examples/invoice-exception/README.md) | Shows a controlled-write system and an end-to-end hybrid FDE walkthrough | In-memory teaching implementations and regression surfaces |
| [`patterns/`](patterns/pattern-catalog.json) | Records patterns, anti-patterns, controls, evidence, and review dates | Machine-readable decision catalog |
| [`library/`](library/00-start-here.md) | Explains design decisions, implementation sequence, and failure modes | Human-readable guidance |
| [`operations/`](operations/README.md) | Defines release, telemetry, service objectives, incident response, and change | Operating contract |
| [`research/`](research/README.md) | Records dated sources, portable findings, and caveats | Evidence for claims that can change |
| [`docs/maintainers/`](docs/maintainers/repository-maintenance.md) | Defines repository stewardship and release maintenance | Internal maintainer runbook |
| [`scripts/`](scripts/validate-repository.mjs) | Validates repository-wide structure and cross-references | Automated repository guardrail |
| [`tests/`](tests/) | Exercises shared schemas, path containment, and Markdown behavior | Repository-level regression suite |

## Route by task

| Task | Read next | Expected result |
| --- | --- | --- |
| Lead an FDE or internal delivery engagement | [FDE playbooks](playbooks/README.md) → current lifecycle stage → required templates | Evidence-backed decisions from qualification through business-owned production operation |
| Build shared applied-AI capability | [FDE and applied AI engineering synthesis](library/10-fde-and-production-agent-synthesis.md) → current lifecycle stage → relevant reusable artifact | A deliberate boundary between workflow-specific delivery, reusable product/platform capability, and sanitized field learning |
| Select a workflow | [Discovery and Value](playbooks/01-discovery-and-value.md) → [Start Here](library/00-start-here.md) → [discovery pack](templates/fde-discovery-pack.md) → [workflow charter](templates/workflow-charter.json) | Observed workflow, owner, baseline, accepted outcome, verifier, value hypothesis, and risk ceiling |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [davidahmann/fde-guide](https://github.com/davidahmann/fde-guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
