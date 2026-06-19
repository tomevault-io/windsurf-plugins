---
trigger: always_on
description: Make every code submission (commit, branch, PR) easier to read for the next person picking it up — human or AI. Forces per-surface changelog entries (one append-only file per page/component/server module/db table/integration/script, not one undifferentiated git log) and (when UI changed) a verified demo recording with both DOM checks and Gemini video analysis. Use any time you commit, push, open a PR, hand off a branch, or finish a task that touched user-facing code.
---


# Easier-to-read submissions

A code submission — a commit, a branch, a PR, a hand-off — that the next person can read without spelunking. The next person might be your reviewer, your future self three months from now, the engineer you're handing the project to, or the next AI agent picking up the branch. They all benefit from the same protocol.

You are about to commit, push, open a PR, or hand off a branch. Before you call the change "done," you owe two artifacts that make the submission readable:

1. **Per-surface changelog entries** — one entry in each lane file for every surface your diff touched. Append-only, dated, cross-linked.
2. **(When the change touched a screen the demo asserts)** A re-recorded, verified demo MP4 + GIF + evidence JSON. Both DOM checks and a Gemini video pass must succeed before you push.

Both halves exist because **a code diff alone is illegible** to the next person on the branch — human or AI. Without per-surface lanes, the next contributor has to read the whole `git log` to understand what one screen used to look like. Without a verified demo, your "this works" claim is unfalsifiable.

This protocol came out of the SitFlow → Jaynee handoff — see [`https://github.com/HomenShum/easier-to-read-submissions`](https://github.com/HomenShum/easier-to-read-submissions) for the public skill repo and the original use case.

---

## Phase 1 — Per-surface changelog lanes (always)

### What a "lane" is

For every user-facing surface in the repo (page, component, server module, db table, integration, script), there's an append-only changelog file at `CHANGELOG/<category>/<slug>.md`. The repo's top-level `git log` is one undifferentiated stream — useful for "what shipped this week," useless for "what has the Inbox screen looked like over time." Per-surface lanes solve that.

### When you change code

1. **Identify every surface your diff touched.** Look at `git status` + `git diff --stat`. A typical change touches 1-3 surfaces. Multi-surface = multiple lane updates.

2. **For each touched surface**, find its lane file (e.g., `CHANGELOG/components/CareCard.md`). If the lane doesn't exist, **create it** using `templates/lane.md` from this skill. Slug naming: replace `/` with `-`, drop extension, drop angle brackets and parens. Examples:
   - `components/CareCard.tsx` → `CHANGELOG/components/CareCard.md`
   - `app/(tabs)/index.tsx` → `CHANGELOG/pages/tabs-index-inbox.md`
   - `server/_core/index.ts` → `CHANGELOG/server/_core-index.md`
   - Database table named `care_rules` → `CHANGELOG/db/care_rules.md`

3. **Prepend a new entry at the top** of each lane file (right after the file header, before the previous most-recent entry). Use the entry template — every lane file ends with one. **Never delete or rewrite old entries.** The audit trail is the whole point.

4. **Cross-link multi-surface changes via `**Touches**:`** — same date, same commit hash on every entry. Each entry lists the other lanes it cross-references. Bidirectional linkage means the next agent can grep into any lane and find every related change.

### Entry format

```md
## YYYY-MM-DD — Short imperative title
What changed and **why** (1-3 sentences, written for the next person who has to maintain this — not for you, the original author). Mention any user-visible effect.
**Commit**: `<7-char sha>`. **Author**: <name>.
**Touches**: `<other CHANGELOG files affected>` (omit if none)
```

Date is `YYYY-MM-DD` (drop time and timezone). Title is imperative ("Add toast on save" not "Added a toast on save"). The 1-3 sentence body answers the next maintainer's question — what + why + user-visible effect. Anything longer belongs in the commit body, not the lane.

### What does NOT get an entry

- Pure formatting fixes / lint runs / typo corrections
- Lockfile bumps from a deps install
- Generated-file regenerations (drizzle migrations, pnpm-lock changes)

If in doubt: would the next maintainer care about this change three months from now? If no, skip the entry.

### CHANGELOG/README.md and TEMPLATE.md

The first time you set up the lanes in a repo, scaffold:
- `CHANGELOG/README.md` — master index organized by category
- `CHANGELOG/TEMPLATE.md` — format spec (this section, basically)

Both ship in `templates/` of this skill. Copy them, then fill in the index with the lane files specific to your repo.

### Bootstrapping an existing repo (one-time)

If a repo doesn't have lanes yet, **dispatch parallel subagents to read `git log --follow` per file and write entries from the commit bodies**. Don't invent entries — only write entries for commits that actually touched the file. See `templates/bootstrap-prompt.md` for the prompt to give each subagent. Slice by category (pages / components / server / db / integrations / scripts) so 4-6 agents can run in parallel without overlap.

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HomenShum/easier-to-read-submissions](https://github.com/HomenShum/easier-to-read-submissions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
