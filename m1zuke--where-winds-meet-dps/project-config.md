---
trigger: always_on
description: Always-on guardrails, plus a router to the detail. Keep this file **short**: if
---

# CLAUDE.md — engine conventions

Always-on guardrails, plus a router to the detail. Keep this file **short**: if
a section here grows past a few lines, it belongs in the topic file instead.

## Docs are implementation rules — the gate on editing them

> **Read this before opening any file in `docs/`. A docs edit that fails this
> gate is a defect, not a contribution.**

`docs/*.md` say **how a thing must be implemented**. They do not describe how the
code works — the code does that, and a prose copy of it rots. Every statement is
a rule an implementer must satisfy, or an external constraint the code cannot
carry.

**The gate: which rule changed?** A commit may touch `docs/` only if the same
commit changes a system contract — a type, a schema field, an engine rule, an
invariant, a convention. If you cannot name the rule that changed, do not touch
`docs/`.

None of these earn a docs edit:

- adding or changing a skill, buff, debuff, mechanic, class, rotation, inner way
- implementing something the docs already state as a rule
- recording that work happened, what it used to be, or who decided it
- an example, a worked walkthrough, or a status / coverage note

1. **Nothing content-specific — `docs/` or the wiki.** No skill, buff, debuff,
   inner way or gear set name or id; no coefficient; no frame count. Examples use
   placeholders. Class names appear only in `docs/CLASSES.md`'s implemented
   table, `docs/TESTING.md`'s scoping rule and `docs/REFERENCE-DATA.md`.
   Mechanically enforced by `tests/data/docsStayGeneral.test.ts`.
2. **Every sentence must hold for every class and skill.** If it is only true of
   one it is not a rule, and it belongs in no docs file at all. Genuinely complex
   per-skill logic gets a short comment in the `.ts` that defines the skill —
   nothing else, nowhere else.
3. **No descriptions of how code works.** No module tours, no folder trees, no
   call-order walkthroughs, no "X then calls Y". If a reader gets it by opening
   the file, it is noise.
4. **No history.** No dates, no decision provenance, no changelog sections.
5. **Adding does not add prose.** If your change made a docs file longer without
   changing a rule, delete what you added.

Every section should read like a checklist: imperative, checkable, no story.

## Read this first, by topic

| working on                                                         | read                     |
| ------------------------------------------------------------------ | ------------------------ |
| damage math — the formula chain, stat layer, calculation rules     | `docs/CALCULATION.md`    |
| a skill, trigger, buff or debuff **data model**                    | `docs/TIMELINE.md`       |
| whether a mechanic is a stat buff or a skill buff                  | `docs/BUFFS.md`          |
| a class, skill/buff data file, or anything that mints an entity id | `docs/CLASSES.md`        |
| `src/ui/**`, `App.tsx`, or `dpsWorker.ts`                          | `docs/UI.md`             |
| writing a localStorage migration                                   | `docs/MIGRATIONS.md`     |
| adding or changing tests                                           | `docs/TESTING.md`        |
| dev-only reference material outside `src/`                         | `docs/REFERENCE-DATA.md` |
| user-visible text, a locale or the translation catalogue           | `docs/I18N.md`           |

## Adding something new — start from the wiki how-to

**Adding** a thing has a step-by-step page; the `docs/` files above explain the
system it plugs into. Read the how-to **first** — it is the ordered file list,
the wiring, the tests to update and the migration question, in one place.

| adding                       | read                          |
| ---------------------------- | ----------------------------- |
| a class or spec              | `How to Add a Class`          |
| a skill                      | `How to Add a Skill`          |
| a rotation                   | `How to Add a Rotation`       |
| a buff, debuff or DoT        | `How to Add a Buff or Debuff` |
| an inner way (mind method)   | `How to Add an Inner Way`     |
| a changelog entry, releasing | `src/changelog/README.md`     |

The changelog how-to is the one that lives in the repo rather than the wiki: it
is read while editing the folder it documents.

The wiki is the [project wiki](https://github.com/M1zuke/where-winds-meet-dps/wiki),
cloned beside this repo at `../where-winds-meet-dps.wiki` — read the `.md` files
there directly. It also carries `Development Setup`, `Architecture Overview`,
`Project Conventions`, `Damage Calculation`, `Testing Guide`,
`Saved Profile Migrations` and `Glossary`.

Two rules keep it trustworthy:

1. **One source per fact.** A rule, a number, a formula and a field name live in
   `docs/` and nowhere else. The wiki carries the ordered procedure and the _why_,
   and links to the `docs/` rule instead of restating it — so the two can never
   disagree. The gate above binds the wiki too: no content names, no examples
   built on a real skill.
2. A change that invalidates a how-to updates that page **in the same piece of
   work**, as its own commit in the wiki clone. A stale how-to is worse than a
   missing one.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [M1zuke/where-winds-meet-dps](https://github.com/M1zuke/where-winds-meet-dps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-14 -->
