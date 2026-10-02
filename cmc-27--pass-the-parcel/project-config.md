---
trigger: always_on
description: > **What this repo is.** Pass the Parcel is a template whose **goal** is **stateless, multi-agent planning & execution** — the 10-phase parcel pipeline with four hard gates and independent review. Its **principal instrument** is an **agent-first wiki** — a governed, grounded knowledge base (deterministic linter + Grounded Claims + drift automation) that keeps an agent's context cheap and honest — supported by **cache-first context**, **Managed Simplicity** and **deterministic guardrails**. This 
---

# Pass the Parcel — Agent Entry Point

> **What this repo is.** Pass the Parcel is a template whose **goal** is **stateless, multi-agent planning & execution** — the 10-phase parcel pipeline with four hard gates and independent review. Its **principal instrument** is an **agent-first wiki** — a governed, grounded knowledge base (deterministic linter + Grounded Claims + drift automation) that keeps an agent's context cheap and honest — supported by **cache-first context**, **Managed Simplicity** and **deterministic guardrails**. This repo is the **template, not an app**: the app-facing content under `.wiki/` documents the pattern satellites fill in. See [`OPERATING-PRINCIPLES.md`](OPERATING-PRINCIPLES.md); maturity is tracked per axis in [`.devops/backlog/MATURITY.md`](.devops/backlog/MATURITY.md).

This repository is configured with a structured documentation library in **`.wiki/`** designed to serve as the single source of truth for the codebase, architecture, state management, and user interfaces.

### Documentation Structure
- **`.wiki/`** — Architecture knowledge, design system, features, and technical specs
- **`.devops/plans/`** — **Plans.** Claimed / in-flight `*-plan.md` only (`PHASE_1`+); template at `template-plan.md`
- **`.devops/sprints/`** — **Sprints (optional).** Active `sprint-{n}-<slug>/sprint.md` + the committed plan queue; indexed by `.devops/backlog/SPRINTS.md`; seeds ship in `.devops/templates/`; adopted by running `@sprint-plan`
- **`.devops/archive/`** — Completed plans (`*-plan.md` at root) + closed sprint records (`sprints/sprint-{n}-<slug>/sprint.md`)
- **`.devops/backlog/`** — **Backlog.** Master queue `backlog-index.md` (Themes table + Triage Panel), theme registers `t{n}-<slug>-backlog.md`, and parked `<code>-<slug>-backlog.md` plans (`claim_status: QUEUED`); commit via `@sprint-plan`, claim into `.devops/plans/`
- **`.devops/logs/`** — Agent changelog, version history
- **`.devops/skills/`** — All skills (SKILL.md per folder), loaded via `opencode.json` `skills` — the V2-native **flat array** (`[".devops/skills"]`). The live config is deliberately **mixed-dialect**: `skills` is V2-native while `agent` / `prompt` / `permission` stay V1-by-normalisation (V2 loads them by normalising, so they are left exactly as authored). JSON carries no comments, so the seed's own `_comment` records this where a bootstrapping maintainer meets it first.
- **`.devops/agents/`** — VS Code custom agents: `parcel.agent.md` + `parcel-sprint.agent.md` (orchestrators; `parcel-sprint` is the locked batch host) + `wiki-writer.agent.md` (selectable; `wiki-writer` is also subagent-invocable), `ptp-*.subagent.md` + `wiki-verifier.subagent.md` (subagents; `ptp-parcel-fast` is the hidden per-plan fast runner, spawned only by `parcel-sprint`)
- **`.wiki/rules/`** — Wiki governance layer — numbering, naming, frontmatter, doc-structure, link-hygiene, structure manifest + deterministic linter
- **`.wiki/rules/language/`** — Language governance layer — voice & tone, AI rules, publication rules
- **`.devops/rules/`** — Dev governance layer — agents & skills, plan lifecycle (canonical home of the plan lifecycle: [`.devops/rules/plan-lifecycle.md`](.devops/rules/plan-lifecycle.md))

Instead of searching the entire codebase to understand context, **STOP** and read the localized intelligence hub first.

---

## The Goal

**Pass the Parcel** is this template's purpose: turn a feature request into a reviewed, executed, verified change by passing one Markdown plan between specialised agents. It is stateless, independently reviewed, gated (A → B → C → D), and deterministic. The **agent-managed wiki** is the principal instrument; **Cache-first context**, **Managed Simplicity** and **deterministic guardrails** are the supporting instruments. Full statement: [`OPERATING-PRINCIPLES.md`](OPERATING-PRINCIPLES.md).

---

## Managed Simplicity

> **Managed Simplicity.** We do one thing, we do it well, and we do it fast. Structure must earn its cost: one canonical home per rule, one deterministic check per invariant, no surface that has stopped paying for itself. We do not build machinery for edge cases — we remove or accept them. Depth (the wiki, the pipeline) is bought for outcomes. See `.devops/rules/managed-simplicity.md`.

---

## Design & Scope Notes

> [!NOTE]
> **This repo is the template, not an app.** The task-lookup rows that reference `src/components/ui`, screens, database queries, and CSV parsing are **satellite-facing examples** — they apply in workspaces that contain an application source tree. In this template they document the pattern satellites follow; there is no frontend or database here to edit.
>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CMC-27/pass-the-parcel](https://github.com/CMC-27/pass-the-parcel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
