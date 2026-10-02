---
trigger: always_on
description: **Purpose:** the repository-wide operating contract for AI agents and automated contributors. Read this before modifying any file. More specific scoped instructions, when present, refine this contract for their subtree.
---

# AGENTS.md — AI Agent Contract

**Purpose:** the repository-wide operating contract for AI agents and automated contributors. Read this before modifying any file. More specific scoped instructions, when present, refine this contract for their subtree.

## What this repository is

`agent-toolkit` is the capability distribution layer for reusable AI-agent skills, personas, loops, MCP templates, products/packs, target profiles, the native V CLI, and Agent Toolkit Desktop. Changes can affect multiple downstream coding assistants and packaging channels.

Operate precisely: use canonical sources, preserve public/vendor-neutral boundaries, cite evidence, and validate before finalizing.

## Canonical document map

Use the narrowest current source of truth instead of old issue prose or historical comments.

| Concern | Canonical source |
|---|---|
| Public concept model / boundaries | [`docs/CONCEPTS.md`](docs/CONCEPTS.md) |
| Architecture decisions | [`docs/adrs/`](docs/adrs/) |
| Skill authoring/integration | [`docs/SKILL_INTEGRATION_CHECKLIST.md`](docs/SKILL_INTEGRATION_CHECKLIST.md), [`docs/UPSTREAM_VS_FIRST_PARTY.md`](docs/UPSTREAM_VS_FIRST_PARTY.md) |
| Agent personas | [`docs/HOW_TO_ADD_AGENT.md`](docs/HOW_TO_ADD_AGENT.md) |
| Loop templates | [`docs/HOW_TO_CREATE_LOOP.md`](docs/HOW_TO_CREATE_LOOP.md), [`schemas/loop.schema.json`](schemas/loop.schema.json) |
| V development | [`docs/HOW_TO_DEVELOP_V.md`](docs/HOW_TO_DEVELOP_V.md) |
| Desktop product | [`docs/desktop/PRODUCT_VISION.md`](docs/desktop/PRODUCT_VISION.md) |
| Desktop interaction model | [`docs/desktop/UX_ARCHITECTURE.md`](docs/desktop/UX_ARCHITECTURE.md) |
| Desktop visual design | [`docs/desktop/DESIGN.md`](docs/desktop/DESIGN.md) |
| Desktop semantic world (primary spatial home) | [`docs/desktop/SEMANTIC_WORLD.md`](docs/desktop/SEMANTIC_WORLD.md), [`docs/adrs/ADR-034-semantic-world.md`](docs/adrs/ADR-034-semantic-world.md) |
| Desktop journeys / coverage | [`docs/desktop/USER_JOURNEYS.md`](docs/desktop/USER_JOURNEYS.md), [`docs/desktop/WORKFLOW_COVERAGE.md`](docs/desktop/WORKFLOW_COVERAGE.md) |
| Desktop truth ledger | [`docs/desktop/TRUTH_LEDGER.md`](docs/desktop/TRUTH_LEDGER.md) |
| Desktop visual acceptance | [`docs/desktop/VISUAL_QA.md`](docs/desktop/VISUAL_QA.md) |
| Desktop backlog evidence (HISTORICAL AUDIT — archived 2026-09-10, do not update) | [`docs/desktop/BACKLOG_AUDIT.md`](docs/desktop/BACKLOG_AUDIT.md) |

Historical ADRs and GitHub issues are evidence, not automatic implementation authority. Reconcile them with current code and governing contracts before acting.

## Instruction precedence inside this project

Repository work should follow, in order:

1. explicit current task requirements;
2. this `AGENTS.md` and any scoped descendant instructions;
3. current ADRs and canonical domain contracts;
4. current product/UX/design contracts;
5. current verified code/tests/runtime behavior;
6. GitHub issue implementation details;
7. historical plans/comments.

This project ordering does not override system/developer/runtime instructions from the agent platform itself.

## Global invariants

### Always

- Use English for repository content, commits, issues, and PR descriptions.
- Keep secrets, credentials, tokens, private-company content, and private customer data out of this public repository.
- Prefer canonical catalogs/manifests/core operations over duplicated hardcoded rosters.
- Cite the file/contract behind repository conventions when making architectural claims.
- Inspect current code and generated sources before claiming compatibility or completion.
- Work in focused branches/PRs for implementation unless the maintainer explicitly requests another workflow.
- Preserve user-owned files and report partial failure honestly.

### Never

- Manufacture plausible production state, progress, health, receipts, provenance, compatibility, costs, activity, or Git history.
- Hand-edit generated catalogs/manifests when a generator owns them.
- Commit credentials or real secret values to MCP templates/examples.
- Claim tool compatibility without evidence from metadata, tests, or explicit supported behavior.
- Add private-organization-specific content to first-party public capabilities.
- Add third-party npm/GitHub/URL packs to `distributions/products.yaml` or generated plugin surfaces; follow the third-party boundary in `docs/CONCEPTS.md`.
- Bypass branch protection or weaken validation merely to land a change.

Unknown, unavailable, empty, and unverified are valid states. Fabricated-but-plausible is not.

## Repository ownership model

Key source areas:

- `skills/` — first-party reusable capability sources (`SKILL.md`)
- `agents/` — tool-agnostic personas (`AGENT.md`)
- `loops/` — loop templates (`loop.yaml`)
- `mcp/` — provider registry/templates
- `profiles/` — target-specific adapters/overlays
- `packs/` — solution/workflow packs
- `capabilities/` — declarative capability registries (targets, skills, hooks); canonical source for supported-target and capability truth
- `distributions/` — product compiler input

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ulises-jeremias/agent-toolkit](https://github.com/ulises-jeremias/agent-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
