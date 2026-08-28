---
trigger: always_on
description: Make code correct, maintainable, and verified — and spend effort in proportion
---

# Claude Coding Agent Instructions

## Core philosophy

Make code correct, maintainable, and verified — and spend effort in proportion
to the change. Process exists to protect large or risky work, not to tax small
work. A one-line answer should take one line.

---

## Scale the process to the change

Decide which tier a task falls in before starting. When unsure, pick the lower
tier and escalate if the work turns out bigger than it looked.

| Tier | Examples | What's required |
|---|---|---|
| **Trivial** | Answer a question, give a command, rename variables, fix a typo, tweak a config value, explain existing code | Just do it. No memory reads, no notes, no docs, no STATUS. Reply in a sentence or two. |
| **Normal** | Bug fix, new function, small feature, single-module change | Understand → implement → test → short report. Update docs only if behaviour or the public API changed. |
| **Significant** | New feature, refactor touching >5 files, breaking change, new/moved directory, anything that changes how the repo is laid out | Full workflow below: change doc, README map, `STATUS.md`, memory note. |

**Only write or change tracked docs, `STATUS.md`, and memory when the change is
significant.** The same bar that governs markdown changelogs governs everything
else. Routine work leaves no paperwork behind.

---

## Workflow (normal and significant work)

1. **Understand** — read the relevant code and tests first.
2. **Locate** — decide where new files belong before creating them.
3. **Implement** — clean, focused code; one concern per function.
4. **Test** — write and run tests; fix real failures rather than papering over them.
5. **Document** — inline comments always; a change doc only if significant.
6. **Record** — `STATUS.md` and a memory note only if significant.
7. **Verify** — check the task is actually done before saying so.

For significant work only, start by reading `.claude/memory/practices.md` and
`STATUS.md` so you don't rediscover what a previous session recorded.

---

## Repository organization

### Keep the root clean

Don't create new files in the repo root unless they're one of: `README.md`,
`CLAUDE.md`, `STATUS.md`, `LICENSE`, `.gitignore`, `.env.example`, the language
manifest/lockfile, or a linter/CI config.

Everything else — scripts, notes, experiments, sample data, generated output —
goes in a subdirectory.

### Canonical layout

Follow the repo's existing convention if it has one. Otherwise:

```
repo/
├── README.md          # entry point + repository map
├── STATUS.md          # project state (significant changes only)
├── src/<package>/     # source
├── tests/             # mirrors src/ one-to-one
├── scripts/           # runnable operational scripts
├── docs/              # design docs, ADRs, change docs
├── config/            # config, templates, schemas
├── .claude/memory/    # gitignored agent memory
└── .scratch/          # gitignored throwaway work
```

### Choosing a location

1. Search for existing files doing a similar job and put yours next to them.
2. Check the repository map in the root `README.md`.
3. Match sibling naming (`payment_service.py`, not `paymentSvc.py`).
4. If nothing fits, don't invent a top-level directory — propose it in your
   response with a one-line rationale and prefer nesting under an existing one.

### Naming

Descriptive and durable. Never `new_`, `old_`, `final_`, `v2_`, `_updated`,
`_fixed`, `copy_`, or `temp_` in a committed filename — version control handles
versions.

### Temporary files

Exploratory scripts and debugging harnesses go in `.scratch/` (gitignored), and
are deleted or promoted to `scripts/` before you finish. Generated artifacts
are gitignored, never committed.

### Moving files

Update every reference in the same change: imports, config, CI, docs, README
map. Prefer moving over duplicating.

---

## README

The root `README.md` should answer, within one screen: what is this, how do I
run it, where do I start reading, where does new code go.

- **Map directories, not files.** Describe each directory's purpose and name one
  entry-point file. Exhaustive file listings go stale immediately.
- **Structural change ⇒ README update, same commit.** Adding, removing,
  renaming, or repurposing a directory means updating the map. Nothing else
  requires touching the README.
- Give a directory its own `README.md` only if it has many files and a
  non-obvious purpose.
- If you notice the README contradicting reality, fix it and flag it.

---

## Agent memory

`.claude/memory/` is gitignored and never committed — not even when a commit is
explicitly requested. Confirm `.gitignore` covers it before writing there.

```
.claude/memory/
├── practices.md      # verified environment + commands
├── index.md          # one row per task note, newest first
└── tasks/YYYY-MM-DD-<slug>.md
```

### practices.md

The one file worth maintaining routinely. Record only what you **verified in
this repo** — a command goes in after it ran successfully. Fix a wrong entry the
moment you hit it; a stale command here costs every future session.

Keep it under 60 lines: environment, a command table (install / test / lint /
format / types), key conventions, and gotchas. Tables and fragments, no prose.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tpmmthomas/claudemd](https://github.com/tpmmthomas/claudemd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
