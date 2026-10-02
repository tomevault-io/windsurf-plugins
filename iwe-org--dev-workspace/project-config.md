---
trigger: always_on
description: You are operating a **dev workspace**: a markdown knowledge graph that is a
---

# Agent operating manual

You are operating a **dev workspace**: a markdown knowledge graph that is a
software project's memory and system of record. The division of labor:

- **Code** lives in the project's own repository. Writing it is your normal work
  — this workspace doesn't change how you code.
- **Project state** lives here, in `data/` — what the product is, how it must
  behave (specs), how it's designed (architecture), what's planned, shipped,
  broken, and released. Every working session must leave a record in the graph;
  this is your memory across sessions. Never keep project state only in
  conversation.

## Start of every session

1. Read `data/product.md`. If it still contains ✏️ placeholders, run the setup
   flow (`.claude/skills/setup/SKILL.md`) before anything else — planning
   without product context is guessing. Note `## Constraints` and
   `## Authoring rules`: they bind everything you write.
2. Check the state of work: active plans under `## Active` in `data/plans.md`,
   and high-priority tasks —
   `iwe find --filter '{stage: planned, priority: high}' --included-by data/backlog -f keys`.

## The operating loop

1. **Pick** the next piece of work (the user's request, an active plan, or the
   backlog head).
2. **Consult** before acting: the relevant `data/spec/` docs (intended
   behavior), `data/architecture/` (design and past decisions), and the
   feature/bug doc the work belongs to. If a plan exists, execute the plan; if
   the work deserves one, write it first (plan skill).
3. **Execute** — implement in the codebase, following the plan's tasks (the
   implement skill keeps checkboxes, anchors, and deviations honest while you
   do).
4. **Record** — write the state back:
   - Idea (not a commitment) → `data/someday/<slug>.md` + link from
     `data/someday.md`.
   - Actionable item → `data/backlog/<slug>.md` (`stage: planned`, priority),
     linked under the priority section of `data/backlog.md`.
   - Work starts → plan skill: `data/plans/YYYYMMDD-<slug>.md` (`created`,
     verified code anchors, `## Spec changes`) + link under `## Active`.
   - Work ships → verify skill green (tasks, requirements, and scenarios checked
     against the code), then ship skill: specs synced first, then `stage: done`
     with `completed`, link moved to `## Done`, feature doc `implemented`,
     inclusion link in `data/releases/unreleased.md`.
   - Plan abandoned → `stage: cancelled`, link moved to `## Cancelled` (it stays
     listed — the record of why is worth keeping).
   - Bug found → `data/bugs/<slug>.md` (Symptom / Reproduction / Root cause /
     Fix, `path:line` anchors) + link from `data/bugs.md`. Fixed →
     `stage: done`.
   - Behavior defined or changed → the matching `data/spec/` doc
     (Requirement/Scenario format); this happens *inside* the ship flow, not as
     an afterthought.
   - Design decision made → `data/architecture/<slug>.md`, including the
     rejected alternatives.
   - Code structure changed (module added, split, or moved) → re-read the code
     and refresh the touched `data/codebase/` docs, bumping their `commit` and
     `verified`. `git log <commit>..HEAD -- <source>` finds the stale ones.
   - Vision insight → `data/concept/<slug>.md`.
   - Task finished → `stage: done` + `completed` on the task doc, link moved to
     `## Done` in `data/backlog.md`.
   - Release cut → ship skill's release mode (rename unreleased, stamp
     version/date, fresh accumulator).
5. **Stamp** — every document you create or meaningfully change gets
   `generated: { by: claude-code/opus-5, at: <ISO 8601 now> }`, a one-sentence
   `description` if it has none, and — when you derived it from code or an
   external page — a `sources` entry naming that path or URL. Whenever you set
   `stage`, derive OKF `status` from the table in `SCHEMA.md` and set or clear
   it in the same edit. If a hub gained or lost a document, update
   `data/index.md`.
6. **Validate & commit** — `iwe normalize`, then `iwe schema validate` must
   pass; commit with a short message describing the state change.

## Conventions

- **Inclusion link** = a markdown link on its own line — it makes the target a
  child in the graph. Hubs (`data/plans.md`, `data/features.md`, …)
  inclusion-link their members; that link, not the directory, is what makes a
  document a plan or a feature. Inline links (inside sentences/list items) are
  soft references for cross-cutting relationships.
- **Dual representation**: a work item's stage lives in frontmatter *and* as its
  link's position in the hub (`## Active`/`## Done`/`## Cancelled` in plans,
  `## High`/`## Done` in backlog). Change both together; every item stays listed
  forever.
- **Stage vocabularies** (schema-enforced, human reference in `SCHEMA.md`):
  plans `done|cancelled` (absent = active, `done` requires `completed`);
  features `proposed|accepted|implemented|deprecated|cancelled`; bugs
  `done|cancelled` (absent = open); releases `released|unreleased`; backlog
  `planned|done`. Reference docs (spec/architecture/concept/someday) carry a
  `type` and no stage; codebase-map docs carry `source` + `commit` + `verified`
  — provenance, not lifecycle.
- **`data/` is an OKF v0.2 bundle** — the graph is portable knowledge any [Open
  Knowledge

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iwe-org/dev-workspace](https://github.com/iwe-org/dev-workspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
