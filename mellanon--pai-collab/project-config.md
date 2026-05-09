---
trigger: always_on
description: You are working in pai-collab, a shared blackboard for PAI community collaboration. Follow these protocols whenever you work in this repository.
---

# pai-collab — Agent Operating Protocol

You are working in pai-collab, a shared blackboard for PAI community collaboration. Follow these protocols whenever you work in this repository.

---

## Self-Alignment Check (MANDATORY)

**After every policy change, issue closure, or SOP modification**, review this file against the current state of all policy documents and SOPs:

1. Read `TRUST-MODEL.md`, all files in `sops/`, `CONTRIBUTING.md`, and `REGISTRY.md`
2. Check whether this file (CLAUDE.md) accurately reflects the current procedures, trust zones, and protocols
3. If any procedure has changed, update this file to match
4. If this file references an SOP or policy that has been modified, align the reference

**This file is the codified version of the standard operating procedures.** If an SOP says one thing and this file says another, that is a bug. Fix it immediately.

**The cascade works both ways:**
- Policy/SOP change → check this file → update if misaligned
- This file change → check policies/SOPs → update if misaligned

---

## Artifact Schemas (MANDATORY)

All artifacts in this repository follow canonical schemas defined in `CONTRIBUTING.md`. When creating or modifying any artifact, follow the schema exactly.

| Artifact | Schema Location | Triggers for Update |
|----------|----------------|-------------------|
| `PROJECT.yaml` | CONTRIBUTING.md → "PROJECT.yaml Schema" | Creating a new project, changing project status, adding contributors |
| `JOURNAL.md` (project) | CONTRIBUTING.md → "JOURNAL.md Schema" | After every commit that changes project files, after actioning a project issue |
| `JOURNAL.md` (root) | CONTRIBUTING.md → "JOURNAL.md Schema" | After every commit that changes governance files, after actioning a governance issue |
| Project `README.md` | CONTRIBUTING.md → "Project README.md — Minimum Content" | Creating a new project, significant project milestone |
| `REGISTRY.md` entries | CONTRIBUTING.md → "REGISTRY.md Entry Format" | New project registered, project status changes, new agent joins |
| `CONTRIBUTORS.yaml` | TRUST-MODEL.md → "Two-Level Scoping" | New contributor promoted, trust zone changed, profile updated |
| `STATUS.md` | — (root-level project overview) | New project added, project phase changes, contributor promoted or added |
| SOPs | CONTRIBUTING.md → "SOP Format Guide" | Creating or modifying an SOP |
| Governance reviews | `reviews/README.md` | Trust model audits, documentation audits, cross-project reviews |

**Key rules:**
- `PROJECT.yaml` status must use canonical lifecycle values: `proposed`, `building`, `hardening`, `contrib-prep`, `review`, `shipped`, `evolving`, `archived`
- **Repository license is AGPL-3.0.** All contributions to pai-collab (SOPs, governance docs, tooling) are under AGPL-3.0. PRs constitute acceptance of this license (DCO model — see CONTRIBUTING.md)
- `PROJECT.yaml` must include a `license` field with an accepted SPDX identifier: `MIT`, `Apache-2.0`, `BSD-2-Clause`, `BSD-3-Clause`, `CC-BY-4.0`, `AGPL-3.0`. CC-BY-4.0 is accepted for documentation/specification projects. AGPL-3.0 is accepted for infrastructure tooling where network-use protection is required (prevents cloud extraction without contribution). Reject PRs that omit this field or use other copyleft licenses (GPL, LGPL, MPL)
- `REGISTRY.md` status must match `PROJECT.yaml` status — REGISTRY.md is the index, PROJECT.yaml is the source of truth
- `JOURNAL.md` phase values must match the lifecycle: Specify, Build, Harden, Contrib Prep, Review, Release, Evolve
- When a project's status changes, update `PROJECT.yaml`, `REGISTRY.md`, AND `STATUS.md` in the same commit

---

## Journaling Protocol

**After every commit**, update the relevant `JOURNAL.md` following the schema in CONTRIBUTING.md:
- Add a new entry at the top (reverse chronological)
- Required fields: date, author, phase, status, what happened, what emerged
- Include issue references and what follow-up was created
- If a commit spans multiple projects, update each project's journal
- Each entry must be self-contained — readable without context from other entries

**Where to journal:**
- **Project-specific changes** → that project's `JOURNAL.md` (e.g., `projects/signal/JOURNAL.md`)
- **Governance changes** (SOPs, TRUST-MODEL.md, CLAUDE.md, CONTRIBUTING.md, README.md, STATUS.md, CONTRIBUTORS.yaml) → `JOURNAL.md` at the repo root

**After actioning an issue**, add a journal entry documenting what happened and what emerged — not just the change, but the reasoning and any insights. This applies to both project and governance issues.

---

## Issue Protocol

**All non-trivial changes must be tracked by an issue.** Issues are the unit of traceability — they connect intent (why), scope (what), and evidence (commits, journal entries). Without an issue, work is invisible to other agents and contributors.

### Issue-First Workflow

1. **Before starting work** — Ensure an issue exists. If not, create one.
2. **Label the issue** — Apply scope, type, and priority labels (see below).
3. **Comment that you're working on it** — So other agents don't duplicate effort.
4. **Commit with references** — Use `closes #N` or `partial #N` in commit messages.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mellanon/pai-collab](https://github.com/mellanon/pai-collab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
