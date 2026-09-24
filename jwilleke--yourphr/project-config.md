---
trigger: always_on
description: <!-- KIT:START v1.13.0-3-g9e83986 — managed by mjs-project-template; edit below the KIT:END marker -->
---

<!-- KIT:START v1.13.0-3-g9e83986 — managed by mjs-project-template; edit below the KIT:END marker -->
## Agent Kit Protocols

This section is __managed by the kit__ (`install-kit.sh`) — it is identical across repos. Put repo-specific context __below the `KIT:END` marker__; do not edit here.

The heading above names the kit on purpose. It used to read `Agent Context & Protocols`, which is the
same wording a repo naturally picks for its own agent section below `KIT:END` — two identical `##`
headings in one file, and `markdownlint` MD024 fails on it. The kit owns one heading string in every
repo that installs it, so that string says whose it is.

### Don't Repeat Yourself

Two halves of one rule. Both fire before you write anything.

- __Knowledge: one representation.__ Every fact — a rule, a decision, a version, a list — has exactly one authoritative home. Point at it; never restate it. A second copy is not redundancy, it is a future contradiction: the copies drift, and nothing tells you which one is current. Before writing a fact down, find where it already lives. If it lives in two places already, that is a defect worth fixing, not a pattern to follow.
- __Work: check it is not already done.__ Before starting, read the `▶ Resume here` block in `TODO.md`, recent `git log`, and the related GitHub issue. Repeating finished work is the most common avoidable mistake.

The long form is the operator's __Do NOT Repeat Yourself (DRY)__ page. It lives on a private wiki, so it is named here, not linked. It is not copied here on purpose — copying it would be the very defect this protocol names. The kit owns this paragraph; the page owns the rest.

### Session continuity

- Before starting, read the `▶ Resume here` block at the top of `TODO.md` (committed, so it syncs across machines) and recent `git log`. That is where the last session left off — repeating finished work is the most common avoidable mistake.
- Commit a chunk of work with `/session-commit`: commits code + `TODO.md`, appends a journal entry to `private/project_log.md` (the log is never committed).
- Run `/pstatus` often (after every `/session-commit`): it ranks open work and recommends the next step.
- End a session with `/wrap`: commits anything outstanding, refreshes the `▶ Resume here` pointer, and reports whether it is safe to shut down the editor.

### Priorities — GitHub labels are the source of truth

Priority labels are mutually exclusive and mean:

- `P0` — __Broken. Stop all work and fix it.__ (production down / blocked / security breach)
- `P1` — __Delivers value to the mission.__
- `P2` — __Nice to have.__
- `deferred` — consciously postponed; `needs-triage` — awaiting a priority decision.

Then:

- Security comes first. Scanner alerts (Dependabot / code-scanning / GitGuardian) become issues labeled `security` + a graded priority: critical/high → `P0`, medium → `P1`, low → `P2`.
- `TODO.md` = a `▶ Resume here` block (maintained by `/wrap`) on top, then priority bands that `/pstatus` regenerates from the labels. Do not hand-edit the bands.
- The two halves have one writer each and a deliberate handover: `/wrap` writes the resume pointer at session end, `/context` reads it at session open, and the first `/pstatus` of the session __removes__ it — by then you have already resumed, so it has served its purpose. A bands-only `TODO.md` mid-session is expected, not a loss.
- Kit files are overwritten wholesale on every sync — `.claude/commands/*.md`, `utility/sync-labels.sh`, `.markdownlint-cli2.jsonc`. Never add a rule to one of them: it is destroyed at the next sync (the installer now warns, but the rule still goes). A __generic__ rule belongs upstream in [mjs-project-template](https://github.com/jwilleke/mjs-project-template) so every repo gets it. A __repo-specific__ note about a command — a package manager the kit does not name, a scanner only this repo has — goes in `.claude/commands/<command>.local.md`, which the kit never writes, reads, or deletes. Read that file, if present, as part of the command; commit it, so it travels with the repo.
- `TODO.md` holds __no history__ — only what is open right now. Never add "merged since last run", closed/merged counts, a session narrative, a dated changelog, or work from other repos. A closed item just stops appearing; that disappearance is the whole record. Session history goes in `private/project_log.md` via `/session-commit` and `/wrap`, and nowhere else.

### Working agreement

- Think before coding: state assumptions, surface trade-offs, ask when scope is ambiguous.
- Simplicity first: the minimum that solves the problem; nothing speculative.
- Use Conventional Commits for messages.
- Issue decomposition — NEVER put "Steps", "Phases", or numbered sequences inside a single GitHub issue. Break each step into its own issue and link them using GitHub relationships: `closes #N` / `fixes #N` (resolves another), `blocked by #N` (dependency), `relates to #N` (context link). Example: a 3-phase migration = 3 issues with "blocked by" chains, not one issue with Phase headings.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jwilleke/yourphr](https://github.com/jwilleke/yourphr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
