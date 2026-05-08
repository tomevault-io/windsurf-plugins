---
trigger: always_on
description: Auto-run skill bootstrap: index existing skills, detect gaps, search skills.sh candidates (no auto-install by default)
---


# Auto Skill Bootstrap (always)

## When to run

Run **at the start of any non-trivial task** (anything beyond a simple Q&A), and **again** whenever the user adds new requirements/tech constraints.

## Procedure

## Phases + gating (conditional pre-flight)

The agent must reason about tasks in phases, because “just talking” often becomes planning/spec.

## Default stance: engineering-first

Most tasks in this workspace are software engineering. If the task involves building/changing software (code/config/infra),
the agent MUST prioritize engineering domain coverage first, and treat business domains as additive (not a replacement).

**Engineering triggers (any of these => engineering-first applies):**
- user asks to implement/build/modify code, repo, project, integration, API/SDK/client, service, UI, tests, CI/CD, deploy
- stack/framework/runtime is mentioned or implied (e.g. Next.js/React/TypeScript/Python/Node/.NET/etc.)
- the agent is about to produce a plan/spec that will lead to implementation

**Engineering baseline caps (must be included when triggered):**
- Planning/Spec baseline (Gate A add-on): `web-development` OR (`frontend`/`backend` as applicable), plus `architecture`, `api-design`, `security`, `performance`, `design`, `docs`
- Implementation baseline (Gate B add-on): `code-quality`, `testing-unit`, `security`, plus required stack/domain caps (language/runtime, `frontend`/`backend`, `devops`/`ci-cd`, etc.)

Phases:
- **Exploration**: clarify context and constraints (no decisions yet)
- **Planning/Spec**: produce a plan, spec, architecture decision, or best-practice recommendation
- **Implementation**: change code/configs/infra
- **Verification**: tests, QA scenarios, verification steps
- **Delivery**: CI/CD, deploy, release

**Gate A (Planning Gate)** — BEFORE the agent produces any of the following:
- a plan (ordered steps, milestones, checklists)
- a spec/contract (API, schema, DTOs, error model)
- an architecture decision (stack choice, polling vs webhooks, module boundaries)
- “how to do it correctly” best-practice guidance

The agent MUST run bootstrap for **planning capabilities** and must print them to chat:
- current Phase
- planning caps
- plus any domain/stack caps already known at that point (e.g. `backend`, `marketing`, `python`, etc.)

**Gate B (Implementation Gate)** — BEFORE the agent changes any code/config/infra:

The agent MUST run bootstrap for **implementation capabilities** and must print them to chat:
- current Phase
- implementation caps
- plus the domain/stack caps required to implement (e.g. `backend`/`frontend`, language/runtime, `devops`/`ci-cd`, etc.)

**Required chat output (no skipping):**
- `Phases:` list of phases the agent expects to touch (at least Planning/Spec vs Implementation is required).
- `Selected capabilities:` full list to be passed to bootstrap (and/or `other` queries).
- `Bootstrap result summary:` a short summary from `.cursor/skills/auto-skill-bootstrap/state.json`:
  - `caps`, `missing_caps`, `non_allowlisted_only_caps`, `no_candidates_caps`, and whether `adhoc_queries` are in use.

**User choice UX (mandatory):**
- If the agent needs the user to **choose** between options, it MUST use the built-in multi-choice form (tool AskQuestion), not free-form text.
- The choice UI MUST ALWAYS be **multi-select** (even if a single option would normally be sufficient), so the user can select multiple skills when appropriate.
- The AskQuestion **title must always include** a clear marker like `(multi-select)` / `(multiple)` so the user understands they can pick more than one.
- Choices MUST be asked **per capability**, not as a single combined list across multiple capabilities.
- Each per-capability question MUST include an explicit option like `Nothing / install none` to let the user decline. If the user declines for a capability, the agent MUST persist it and MUST NOT ask again for that capability in this task/session:

```bash
python .cursor/skills/auto-skill-bootstrap/bin/auto-skill-bootstrap.py --no-install \
  --cap <cap1> --cap <cap2> \
  --ignore-cap <cap1>
```

1) **Update local skills inventory** (deterministic):

```bash
python .cursor/skills/auto-skill-bootstrap/bin/update-manifest.py
```

**Note on tooling + scope:**
- This workflow uses the **Skills CLI** via `npx skills` (invoked by the bootstrap script).
- Default install scope is **project-local** (no `-g`). Do not use global installs here unless the user explicitly requests global scope.

2) **Pick capabilities** for the task (3–8 max) using:
- `.cursor/skills/find-skills/domains.json` (domain vocabulary)
- `.cursor/skills/auto-skill-bootstrap/capabilities.json` (capability mapping + queries)

**Domain coverage loop (planning + execution):**
- If the plan/execution expands into a **new domain** (new stack, new process area like testing/CI, new business area like marketing/ads), run bootstrap again for the delta capabilities **before** proceeding with that new domain work.
- `domains.json` is a vocabulary/hint, not a hard limit. If the domain does not map well to existing capabilities, use ad-hoc search:

```bash
python .cursor/skills/auto-skill-bootstrap/bin/auto-skill-bootstrap.py --no-install \
  --cap other \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SteelMorgan/1c-ai-sandbox-client-server](https://github.com/SteelMorgan/1c-ai-sandbox-client-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
