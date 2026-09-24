---
trigger: always_on
description: This repository is a design and verification kit for production AI-enabled systems. It serves applied-AI engineers, product and workflow owners, service operators, and FDEs working inside an organization or with a customer. Optimize for an accepted business outcome that can be independently verified, not for agent autonomy, tool count, model novelty, or architectural complexity.
---

# The Applied AI Field Guide: Agent Working Contract

## Mission

This repository is a design and verification kit for production AI-enabled systems. It serves applied-AI engineers, product and workflow owners, service operators, and FDEs working inside an organization or with a customer. Optimize for an accepted business outcome that can be independently verified, not for agent autonomy, tool count, model novelty, or architectural complexity.

Treat an agent as one component option. For each consequential decision, first compare deterministic code, optimization, classical ML, retrieval, a foundation-model call, a bounded agent workflow, and human review as applicable. Select the smallest sufficient mechanism and preserve the authority, evidence, cost, fallback, and retirement rationale. `ARC-004`, `ARC-005`.

Use [`README.md`](README.md) for the public entry door, [`guide/field-guide-in-five-minutes.md`](guide/field-guide-in-five-minutes.md) for the shortest orientation, the concise [`guide/README.md`](guide/README.md) for the complete human mental model, [`guide/capability-roadmap.md`](guide/capability-roadmap.md) for role and practice orientation, and this file as the working contract for repository navigation and changes.

## Required orientation

Before producing or changing a technical artifact:

1. Inspect [`catalog.json`](catalog.json) to resolve governed artifact IDs and paths.
2. Select the applicable task route below.
3. If the user's job matches a repository skill, read that `SKILL.md` completely and follow its workflow. A skill narrows the route; it does not replace the canonical controls, schemas, or target-system policy.
4. Follow the route's order; load controls, schemas, blueprints, examples, and evidence only when applicable.
5. Read [`README.md`](README.md) when changing public positioning or navigation.
6. Expand context only when the inspected artifact reveals another dependency.

Do not load the entire repository by default. Use one skill or task route, then follow only the direct links needed to complete the work.

## Repository map

| Path | Role | Treat it as |
| --- | --- | --- |
| [`guide/`](guide/README.md) | Gives a five-minute overview, the applied-AI delivery method, and a role and practice roadmap | Human orientation; narrative, not a certification or normative production contract |
| [`catalog.json`](catalog.json) | Lists governed artifacts, types, paths, and tags | Registry; update when a cataloged artifact is added, moved, or removed |
| [`controls/`](controls/control-catalog.json) | Defines production requirements and release gates | Engineering policy normative within this guide |
| [`schemas/`](schemas/README.md) | Defines valid structures for machine-readable artifacts | Structural source of truth |
| [`playbooks/`](playbooks/README.md) | Connects field discovery, value, delivery, adoption, handoff, and operation | End-to-end applied-AI delivery lifecycle |
| [`blueprints/`](blueprints/README.md) | Defines reference components, boundaries, states, failures, and release tests | Architecture starting points, not mandatory frameworks |
| [`solutions/`](solutions/README.md) | Connects business-flow patterns, industry profiles, and horizontal foundations | Design accelerators, not deployable products or release evidence |
| [`templates/`](templates/README.md) | Provides starter design artifacts | Starting material that must be adapted and completed for the target workflow |
| [`examples/`](examples/invoice-exception/README.md) | Shows a complete synthetic invoice engagement, controlled-write, retrieval, and durable-recovery systems, plus an end-to-end applied-AI walkthrough | Worked decision chains, local teaching implementations, and regression surfaces—not customer proof |
| [`patterns/`](patterns/pattern-catalog.json) | Records patterns, anti-patterns, controls, evidence, and review dates | Machine-readable decision catalog |
| [`library/`](library/00-start-here.md) | Explains design decisions, implementation sequence, and failure modes | Human-readable guidance |
| [`operations/`](operations/README.md) | Defines release, telemetry, service objectives, incident response, and change | Operating contract |
| [`research/`](research/README.md) | Records dated sources, portable findings, and caveats | Evidence for claims that can change |
| [`site/`](site/site.config.mjs) | Maps canonical Markdown into the public web guide and provides its minimal UI | Generated discovery layer; never a second content source |
| [`docs/maintainers/`](docs/maintainers/repository-maintenance.md) | Defines repository stewardship and release maintenance | Internal maintainer runbook |
| [`.agents/skills/`](.agents/skills/) | Provides focused applied-AI delivery and engineering workflows | Optional task interfaces over canonical repository artifacts; not authority or runtime capabilities |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [davidahmann/applied-ai-field-guide](https://github.com/davidahmann/applied-ai-field-guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
