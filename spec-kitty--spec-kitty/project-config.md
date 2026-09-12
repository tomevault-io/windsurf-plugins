---
trigger: always_on
description: **Spec Kitty** is a toolkit for Spec-Driven Development (SDD) — clear, actionable specifications ahead of implementation, inspired by GitHub's [Spec Kit](https://github.com/github/spec-kit). **Spec Kitty CLI** bootstraps projects with the framework: directory structures, templates, and AI agent integrations. Every command template leads with a discovery interview; the CLI refuses to create specs or plans until the question set is answered.
---

# Spec Kitty Development Guidelines

**Spec Kitty** is a toolkit for Spec-Driven Development (SDD) — clear, actionable specifications ahead of implementation, inspired by GitHub's [Spec Kit](https://github.com/github/spec-kit). **Spec Kitty CLI** bootstraps projects with the framework: directory structures, templates, and AI agent integrations. Every command template leads with a discovery interview; the CLI refuses to create specs or plans until the question set is answered.

---

## ⚠️ CRITICAL: Load the Project Charter First

**Every LLM agent working in this repository MUST read the project charter at [`.kittify/charter/charter.md`](.kittify/charter/charter.md) at the start of a session, before planning or making changes.**

The charter is the binding governance document. It carries rules that are NOT repeated in this file, including:

- **Governing principles** — single canonical authority, architectural alignment, DDD + tiered rigour, ATDD-first, terminology adherence.
- **Quality & Tech-Debt Standing Orders** — the eight binding practices (adversarial squad cadence, campsite cleaning, mission tracer files, test-remediation/red-first discipline, architectural gate discipline, canonical sources, git/workflow discipline, mission hygiene).
- **Agent operating discipline and collaboration strategy** — model routing, profile-loaded delegation, draft-PR-first, the operator merges.
- **Governance by workflow action** — which rules bind specify/plan/implement/review/merge.

For action-scoped detail, load the doctrine context via `spec-kitty charter context --action <name>` rather than improvising. If the charter and this file ever disagree, the charter wins — flag the drift instead of picking silently.

---

## ⚠️ CRITICAL: Template Source Location

**Edit SOURCE files, NOT agent copies!**

| What | Location | Action |
|------|----------|--------|
| **SOURCE templates** | `packs/built-in/missions/mission-steps/` | ✅ EDIT THESE |
| **Agent copies** | `.claude/`, `.amazonq/`, `.augment/`, etc. | ❌ DO NOT EDIT |

Agent directories are **generated copies** deployed to consumer projects via `spec-kitty upgrade`. Template flow:
```
packs/built-in/missions/mission-steps/{mission_type}/{step_id}/prompt.md  (SOURCE)
    ↓ spec-kitty upgrade
.claude/commands/, .amazonq/prompts/, ... (12 agent dirs + .agents/skills/)  (GENERATED)
```

---

## ⚠️ CRITICAL: Use Canonical Sources, Never Improvise

**Always use the canonical templates, skills, commands, and code surfaces rather than improvising or using older artefacts as examples.**

- Spec/plan/tasks templates come from `packs/built-in/missions/<type>/templates/` (resolved through the charter/doctrine chain) — never copy structure from an older mission in `kitty-specs/`.
- Workflows run through the documented `spec-kitty` CLI commands and the published skills — do not hand-roll equivalents or reconstruct paths the resolver should provide.
- When a canonical command, template, or code surface appears missing or broken, **trace the source and file an upstream gap** — do not silently work around it with an improvised substitute.

**Why:** older missions and ad-hoc artefacts drift from the canonical structure; copying them propagates the drift. The doctrine templates are the single source of truth.

---

## ⚠️ CRITICAL: Team Kitty is Zeitgeist — "sync" is dead

The hosted product is **Team Kitty**; the live transport is **Zeitgeist**, a volatile per-team relay the SaaS provisions and polls. On every lane transition the CLI publishes one **moment** straight to the team's relay (`status/emit.py` → `status/adapters.py` → `status/zeitgeist_bridge.py` → `zeitgeist_client/`), bounded to one request with no queue and no retry, gated only by team membership and repository admission on the SaaS side. The old "sync" transport (daemon, offline queue, per-project consent, `api/v1/sync/*` ingress) was deleted on both sides in August 2026; every remaining "sync" identifier (`SPEC_KITTY_ENABLE_SAAS_SYNC`, `SPEC_KITTY_SYNC_*`, `sync_active()`, `OWNED_SYNC_UNSUPPORTED`) is residue that does **not** gate the moment path. Read [`docs/context/team-kitty.md`](docs/context/team-kitty.md) before touching anything hosted, and never design against or "re-enable" sync.

---

## ⚠️ CRITICAL: Git Workflow — Branches, PRs, and Merges


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spec-kitty/spec-kitty](https://github.com/spec-kitty/spec-kitty) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->
