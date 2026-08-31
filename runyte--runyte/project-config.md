---
trigger: always_on
description: Runyte is a fast modal terminal editor written in Rust, inspired by Helix and
---

# Runyte agent guide

## Project

Runyte is a fast modal terminal editor written in Rust, inspired by Helix and
built on its selection-first model. The editor is the project; keep it first
when weighing a change.

The keymap is close to Helix but has drifted deliberately, most of all in
search and macros. `context/reference/helix-keymap-v1.md` is the register of
record for what matches and what does not; do not describe Runyte as
Helix-compatible without checking it.

Before making substantial changes, read `README.md`, the relevant part of
`docs/user-guide.md`, and `context/README.md`. Then load only the development
records relevant to the task:

- `context/plans/README.md` explains the plan lifecycle and routes to the
  retained architecture records.
- `context/reference/helix-keymap-v1.md` is required when changing editor
  commands, bindings, help, or key hints.
- `context/reference/ui-vocabulary.md` is required when changing buffers,
  panes, lists, pickers, prompts, overlays, or frontend snapshots.
- `context/reference/terminal-compatibility-v1.md` is required when changing
  terminal emulation or PTY behavior.
- `context/reference/startup-performance.md` records measured startup and idle
  cost over time. Consult it when changing startup ordering, document loading,
  syntax parsing, or anything that runs on a timer; the harness that produces
  it is `benchmarks/`.
- `context/issues/` contains open work, `context/issues/deferred/` contains
  confirmed problems awaiting a broader design decision, and
  `context/issues/resolved/` contains searchable diagnoses and regression
  coverage for past fixes.

Historical plans explain decisions but are not a second source of current
behavior. The source, user guide, and current reference documents take
precedence where later work refined a completed plan.

## Terminology

- A **workspace** is the project-root editor scope used in both standalone and
  persistent modes.
- A **persistent session** is the durable local host attachment and retained
  editor state associated with one workspace.
- A **terminal session** is one integrated PTY and child process owned by the
  editor. Use `persistent session` or `terminal session` in prose whenever the
  shorter word would be ambiguous.
- User-facing lifecycle commands use the `session` namespace. Keep workspace
  terminology for project roots, workspace identity and selectors,
  `workspace.mode`, `WorkspaceHost`, and workspace switching.

## Storage boundary

Keep repository development context and runtime workspace state separate:

- `context/` is Git-tracked development context: plans, references, issues,
  and durable project decisions belong here.
- `.runyte/` is Git-ignored editor runtime state used by the optional
  persistent session host and its local transport/lock files.
- Never put runtime state under `context/`.
- Never put credentials, secrets, private model reasoning, or unrestricted
  tool output in tracked context.

`AGENTS.md` is the canonical repository guidance. `CLAUDE.md` points to this
file; do not maintain a second copy.

## Issues

Tracked issues live under `context/issues/`, one Markdown file per issue named
after its topic in `snake_case`: `x_behavior.md`,
`light_and_dark_themes.md`. Files directly in that directory are open; the
ones under `resolved/` are done.

Files under `deferred/` are confirmed problems, but they are also **not open
implementation issues**. Their existing analysis explains why a local fix was
not safe or sufficient. Do not implement or move one in response to a broad
request such as "resolve all remaining issues"; resume it only when the user
explicitly authorizes the deferred work and any required architecture or
design decision has been made.

An open issue is plain prose with no frontmatter or fixed shape. Normalize it
before committing: describe the observed behavior, expected behavior,
constraints, and reproduction in neutral technical language. Preserve exact
keys, examples, errors, and code blocks, but remove conversational requests,
first-person framing, personal paths, and opinions about the implementation
process. An empty file is a placeholder whose report has not been written yet;
ask for it instead of guessing.

Resolving an issue takes two commits:

1. The change itself, with its tests and any `README.md` or
   `context/reference/` updates.
2. A follow-up that moves the file into `context/issues/resolved/` under the
   same name and rewrites it. It is separate because the resolved file
   records the hash of the commit above, which only exists once that commit
   has landed.

A resolved file is frontmatter, then `## Resolution`, then `## Report`:

```markdown
---
title: "One line stating the problem, not the fix"
status: resolved
reported: YYYY-MM-DD
resolved: YYYY-MM-DD
commit: <short hash of the commit that fixed it>
---
```

Resolved records imported at the public-history boundary use
`legacy_commit:` instead. That identifier is provenance from private
development history and does not resolve in the public Git graph. Do not add
new legacy identifiers; fixes made in public history use `commit:` as shown
above.

`## Resolution` opens by naming that commit and its subject, then explains

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [runyte/runyte](https://github.com/runyte/runyte) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
