---
trigger: always_on
description: generates tailored resumes, drafts application answers, and tracks applications —
---

# Job Search Terminal — Agent Instructions

Job Search Terminal is a local-first, dashboard-first job-search command center.
It discovers jobs from ATS APIs, scores them against a career profile using AI,
generates tailored resumes, drafts application answers, and tracks applications —
all running locally on the user's machine with no cloud storage.

## Project Direction

- Claude Code is the primary development and orchestration tool.
- Codex is an approved project collaborator for implementation, review,
  documentation, verification, commits, and pushes when the user requests it.
- `AGENTS.md` is the authoritative agent contract for this repo.
- Detailed project documentation belongs in `docs/`.
- Keep `README.md` short and link to docs instead of duplicating detail.
- After adding or changing functionality, always document it in the same change
  set before considering the work complete. Update user-facing docs, technical
  references, and in-app help as applicable.

## The In-App Help Site Is a Required Documentation Target

`docs/` is written for developers. The in-app help site at `/help` — whose copy
lives in `src/lib/help/content.ts` — is written for job seekers, and it is the
only support surface shipped with the product. There is no support inbox and no
forum. Updating `docs/features.md` while leaving `/help` stale ships a lie.

Update `/help` in the same change set when the change touches any of:

- a new page, tab, nav item, or route
- a renamed button, label, badge, status, table column, or filter
- a new or changed setup step, credential, or environment variable
- what leaves the machine, or any privacy or safety boundary
- a new scan source, import path, or integration
- what an AI feature does, costs, or refuses to do
- a new failure mode a user can hit (this goes in the `troubleshooting` guide)
- anything that makes an existing help sentence false

If none apply, say so explicitly when reporting the work. Silence is not an
answer.

**Read [docs/help-writing.md](docs/help-writing.md) before writing or auditing
any help copy.** It is the full contract — the audience definition, where content
lives, the registry's shape, the voice rules, the trigger list above in longer
form, the procedure for adding a guide, the audit procedure, and the pre-finish
checklist. It is written as instructions to whoever is doing the writing, and it
applies to Codex exactly as it applies to Claude Code.

Both agents use that one file. Claude Code reaches it through a `help-writer`
skill in `.claude/skills/` that does nothing but point at it; Codex reads it
directly from this instruction. Anything you would add to either agent's own
configuration belongs in `docs/help-writing.md` instead, so the two never drift
apart.

The rules broken most often, if you read nothing else: plain language at roughly
a grade 6–8 reading level; name controls exactly as the UI labels them; keep
`steps` (an ordered procedure) distinct from `bullets` (facts); and never pass a
React component reference out of the registry into a client component — it stores
serializable icon *names* and passing the component crashes the build.


---

## Versioning — Bump Every Release, Never Reach 1.0

**Every change set that a user could notice must bump the version and add a
`CHANGELOG.md` entry.** This is a hard rule, in the same class as the
documentation requirements above. Pavel should never have to remember to do it,
or be asked which number to use — decide it from the table and apply it.

The version lives in one place, `package.json`, and the app footer reads it from
there. A change that ships without a bump makes the running app misreport
itself.

### Which number to move

| Bump | When | Example |
|---|---|---|
| **Minor** — `0.11.0` → `0.12.0` | A new capability a user can see: a page, tab, integration, scan source, AI behaviour, or setting. **Also** any database migration, and any change to what leaves the machine. | Added the Cleanup tab; added the Himalayas lane; added the daily update check |
| **Patch** — `0.11.0` → `0.11.1` | A fix or refinement to something that already exists: a bug, wording, layout, accessibility, or performance. | Renamed "Search discover" to "Search for companies"; fixed a stale Dashboard date |
| **Neither** | Documentation, tests, comments, and internal refactors with no user-visible effect. No bump, no changelog entry. | Rewrote a test helper; corrected a stale line in `docs/` |

When one change set contains both a new capability and fixes, it is a **minor**
bump — the highest applicable bump wins, and the fixes ride along in the same
entry.

### Never bump to 1.0.0

The version stays at `0.x` and keeps climbing in the minor position:
`0.9.0` → `0.10.0` → `0.11.0` → `0.12.0`. `0.10.0` is newer than `0.9.0` — the
parts are counted separately, not read as a decimal.

**Only Pavel decides when 1.0.0 happens**, and it is a deliberate promise of
stability rather than a feature count. Never propose it as part of ordinary
work, and never let a bump cross into it.

### What a changelog entry looks like

Add a new section at the **top** of `CHANGELOG.md`, above the previous release:

```markdown
## 0.12.0 — 2026-09-04 — Short name for the release

**Added**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uxdesignlab/job-search-terminal](https://github.com/uxdesignlab/job-search-terminal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
