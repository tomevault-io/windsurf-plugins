---
trigger: always_on
description: Keep this file durable. Derive changing release, provider, branch, and flake
---

# Codewhale agent guidance

Keep this file durable. Derive changing release, provider, branch, and flake
state from the repository, tests, CI, and current issue tracker rather than from
instructions or memory. The nearest scoped `AGENTS.md` adds path-specific rules.

## The ponytail method

From [dietrichgebert/ponytail](https://github.com/dietrichgebert/ponytail) —
"the laziest senior dev in the room." *He says nothing. He writes one line. It
works.* The best code is the code you never wrote.

Before writing code, walk the decision ladder in order and stop at the first
rung that answers:

1. **Does this need to exist?** → Skip it.
2. **Already in this codebase?** → Reuse it.
3. **Stdlib does it?** → Use it.
4. **Native platform feature?** → Use it.
5. **Installed dependency?** → Use it.
6. **One line?** → One line.
7. **Only then:** the minimum that works.

The ladder runs *after* understanding the problem. Lazy about solutions, never
about reading the code first — a short diff written without reading the call
sites is not ponytail, it is a guess.

**Never cut, at any rung:** trust-boundary validation, data-loss handling,
security, accessibility. Brevity is not a reason to drop a guard.

Rung 2 is the one this repository keeps failing. The `model_*` / `*_config` /
`provider_*` grep rule below is rung 2 with a name; so is "one turn loop, one
base prompt". Two more corollaries earned here:

- **An abstraction must delete caller code.** If adopting it is pure
  obligation — required methods, no default bodies that do work — it gets
  built, adopted once, and abandoned.
- **Migrate the last consumer, or do not start.** Framework, one caller,
  ticket the rest, silence the warning: that ships two systems and a comment
  that is no longer true. If the migration will not fit, narrow the slice —
  never the adoption. The standing `#[allow(dead_code)]` count is the running
  receipt; `scripts/check-dead-code-budget.py` prints it.

## Working rules

- Inspect status and existing consumers before editing. Preserve unrelated,
  dirty, and untracked work.
- Before adding a module named `model_*`, `*_config`, `provider_*`, or
  anything that "bridges", "mirrors", or "stages" an existing thing, grep
  for the existing thing and edit it. A new layer must name the predecessor
  it replaces in the module doc; otherwise edit the original.
- Prefer the simplest implementation that preserves observable contracts. A
  rewrite is acceptable when justified by product intent and observed behavior,
  not as a shortcut around understanding existing code.
- Search for behavior and symbols before reviving work from an old branch. If a
  lane is obsolete, preserve its intent and evidence rather than merging stale
  code mechanically.
- A small coherent change may be committed directly to `main` when that checkout
  is current, clean, and owns the affected files. Default to the checkout that
  already exists: when several agents share it, partition by file, stage only
  the paths your slice touched, and retry a commit that fails on `index.lock`.
  A fresh worktree is for conflicting, dirty, stale, or independent lanes
  (see `cw-land`), not for parallel agents on the same lane. Local commit
  permission never implies push, merge, tag, release, or deploy permission.
- When the task is local-only, stay fully offline: no browsing, GitHub or remote
  Git operations, downloads, dependency installation, provider calls, or
  source/diff transmission. Record the missing external receipt and keep working
  locally.
- Public name is **Codewhale**. Compatibility identifiers such as `CodeWhale`,
  `codew`, protocol names, and storage keys change only through an explicit
  migration.
- Keep providers and models first-class and provider-neutral.
- Never rewrite published history, retag a release, force-push a shared ref, or
  publish without explicit authorization. Preserve human contributor credit.
- **Model-visible means logged.** Anything that reaches a model request must be
  reconstructable from the session log, and a new model-visible input needs a
  session event. Live presentation and the persisted record must agree; when they
  disagree the record is right.
- **Misconfiguration fails loud**, at load when it is self-contained, otherwise
  at the earliest point it can be resolved. Never silently skip a missing
  referent.
- **Write down what a design does not do**, beside the behaviour it owns — a
  short known-limitations note in the owning module. A stated limit stops the
  next reader from assuming a capability that was never built.
- **Agents do not comment on issues or PRs** (founder, 2026-09-22). Spend the
  time on code: evidence goes in the commit message and PR body, claims go in
  Linear. Do not reply to review bots or post status, "superseded", or
  "for the record" notes. The one exception is closing or superseding a human
  contributor's PR or issue: one sentence saying why, with the link. The PR and
  issue review workflows are disabled; re-enable one only by founder decision.
- **A user feature lands with its registry row.** A new command, `[features]`
  flag, provider or user-visible feature adds or updates its row in
  `docs/features.toml` in the same change;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [codewhale-hq/Codewhale](https://github.com/codewhale-hq/Codewhale) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
