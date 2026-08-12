---
trigger: always_on
description: > Authoritative index of all skills in this repository. Tools and agents use this
---

# Supreme Team — Skill Manifest

> Authoritative index of all skills in this repository. Tools and agents use this
> file for skill discovery and routing. Paths are relative to the `skills/` directory.

> Installation note: keep the grouped `skills/` source tree intact and install it
> into your tool's configured skill path before invoking any skill. Recommended
> entrypoints are `scripts/install.ps1` on Windows and `scripts/install.sh` on
> macOS/Linux; see `Install.md` for the human and AI-agent installation
> procedure, options, manual fallback, and troubleshooting.

> Discovery prerequisite: assistants do not auto-discover skills from this
> repository checkout unless their configured skill path points here or you have
> copied the tree into `~/.agents/skills/` or `%USERPROFILE%\.agents\skills\`.

> Repository hygiene: `skillset-saves/` is generated Admiral runtime state and
> is intentionally ignored by Git. Do not stage or commit saved runs, locks, audit
> trails, or local scratch directories.

## Entry Routing

`admiral` is the **primary entry orchestrator** — the single front door for the
entire delivery lifecycle (design, build, review, ship, investigate,
checkpoint/resume, gate validation, skill/team creation). Every in-scope request
initiates through `admiral` so that one intake, one persisted run, and one
cross-stage gatekeeper govern the whole pipeline. The binding contract is in
`skills/routing-doctrine.md` and is reinforced deterministically by the
`harness/hooks/user_prompt_submit.py` hook. Standalone Tier-4 tools
(`safety-guardrails/*`, `browser-automation/*`, `release-and-deployment/*`,
`testing-and-qa/*`) run directly, outside this routing.

| Tier | Skills |
|------|--------|
| **Entry orchestrator** | `admiral` |
| **In-scope (must defer to admiral when reached cold)** | `design/commander`, `build/build-management`, `review/code-chief`, `skill-maker`, `investigate`, `session-memory`, `gatekeeper-admiral` |
| **Internal specialists** | every skill under `design/`, `build/`, `review/` not listed above |
| **Standalone tools** | `safety-guardrails/*`, `browser-automation/*`, `release-and-deployment/*`, `testing-and-qa/*` (plus the host `update-config` skill) |

## Admiral Layer (2 skills)

| Skill | Path | Role |
|-------|------|------|
| **admiral** | `skills/admiral/SKILL.md` | Primary entry orchestrator — single front door for the full design→build→review delivery lifecycle, plus investigation, resume, and skill/team creation |
| **gatekeeper-admiral** | `skills/gatekeeper-admiral/SKILL.md` | Cross-stage adversarial validator — validates handoff packages at every major delivery boundary |

## Design Sub-Pipeline (6 skills)

| Skill | Path | Role |
|-------|------|------|
| **commander** | `skills/design/commander/SKILL.md` | Design pipeline orchestrator — delegates to specialists, owns gatekeeper-design cycles |
| **researcher** | `skills/design/researcher/SKILL.md` | Requirements gathering and domain analysis — grounds the design in evidence |
| **planner** | `skills/design/planner/SKILL.md` | Delivery planning with milestones, rollout strategy, decision gates, and risk handling |
| **architect** | `skills/design/architect/SKILL.md` | System architecture (interfaces, API contracts, component boundaries, data flow) **and** owner of the frontend/UI visual design system (shadcn/ui tokens, component template, UI/UX spec, design review) for user-facing surfaces |
| **engineer** | `skills/design/engineer/SKILL.md` | Implementation specification — delivery slices, dependency order, operational constraints |
| **gatekeeper-design** | `skills/design/gatekeeper-design/SKILL.md` | Adversarial quality gate for design deliverables (design→build boundary) |

> The frontend/UI design system that previously lived in a separate `designer`
> skill is now owned by **architect** per `skills/design-doctrine.md`. There is
> no separate `designer` skill and no `tech-stacks/` template library.

## Build Sub-Pipeline (8 skills)

| Skill | Path | Role |
|-------|------|------|
| **build-management** | `skills/build/build-management/SKILL.md` | Build pipeline orchestrator — delegates to specialists, owns gatekeeper-build cycles |
| **bob-the-builder** | `skills/build/bob-the-builder/SKILL.md` | Implements approved scope as production code without placeholders or unowned TODOs |
| **test-builder** | `skills/build/test-builder/SKILL.md` | Builds the automated test surface across intended scope and key failure paths |
| **security-builder** | `skills/build/security-builder/SKILL.md` | Hardens the implementation — unsafe dependencies, insecure patterns, missing controls |
| **cross-check-build-confirm** | `skills/build/cross-check-build-confirm/SKILL.md` | Internal completeness cross-check — verifies the build package is complete and consistent |
| **debugger** | `skills/build/debugger/SKILL.md` | Isolates root causes of a reproduced build-phase failure and returns a bounded fix path |
| **health-check** | `skills/build/health-check/SKILL.md` | Verifies runtime health, startup readiness, and environment dependencies |
| **gatekeeper-build** | `skills/build/gatekeeper-build/SKILL.md` | Adversarial validator of build outputs (build→review boundary) |

## Review Sub-Pipeline (11 skills)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TykoDev/SupremeTeam](https://github.com/TykoDev/SupremeTeam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
