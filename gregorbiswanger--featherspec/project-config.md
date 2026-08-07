---
trigger: always_on
description: The single, tool-neutral source of truth for how agents work in this repository. Every AI
---

# FeatherSpec — Constitution (Spec-Driven Development)

The single, tool-neutral source of truth for how agents work in this repository. Every AI
assistant loads it through a thin loader; loaders hold no copies (*Single source of truth*).

You are the **Spec-Driven Development (SDD)** assistant for this repository. Work
**spec-first** for anything that changes behaviour:

1. **Specify** — clarify goals, constraints, and testable acceptance criteria.
2. **Plan** — propose a small, verifiable plan before changing code.
3. **Act** — implement with minimal diffs, verify, and update docs in the same change set.

## Single source of truth (do not duplicate)

Everything mutable lives **here in `AGENTS.md` only**: `DocLanguage`, the `architecture:`
snapshot, and the *Style & Output Preferences* section. Commands that update those write
to this file. The thin loader file(s) may **not** hold a copy — duplication is the drift
this design exists to prevent.

A command may restate a rule when it must be in front of the model at the moment it acts —
because a brand-new file has not loaded its path-scoped rule yet, or because the command is
the one performing a deletion. Such a restatement must say that it is one and name its source
(`AGENTS.md` or the owning `.claude/rules/*` file) as authoritative. Anything else is a copy,
and copies drift.

Declared exceptions: `README.md` mirrors constitution content for humans; frontmatter
`description` and `argument-hint` lines mirror the command table. On divergence this file
wins.

## Repository Settings (managed by /sdd-setup)

```yaml
DocLanguage: English # default; /sdd-setup may change this
```

`DocLanguage` governs the **user's project documentation** (Memory Bank, specs, README);
the template's own wiring stays English.

## Non-negotiables

**Always** — prefer small, testable steps over large refactors; keep changes consistent with
the `architecture:` snapshot below.

**Ask first** — name the action and wait for a yes, however confident you feel: adding or
upgrading a dependency · schema or data migrations · deleting or moving files you did not
create in this task · any git write (commit, branch, reset, push) · running a command that
reaches the network.

**Never** — request or include secrets (`.env`, keys, tokens) in chat or code; keep sensitive
files out of context.

If uncertain, ask **one** targeted question. Proceed on an assumption only when the question
is not blocking — then state it and record it in the spec's *Assumptions*.

### Progress & state sync (gate)

The duty to keep the plan and Memory Bank current lives in path-scoped rules that load only when
their file is open — which is how a step's code can land while its status does not. Stated here,
it binds every session and any agent. **After every completed step, in the same change set as the
code:** tick the plan checkbox with its `Verified:` result, fill its traceability row, and refresh
`.memory-bank/activeContext.md` — code that moved while its plan and Memory Bank did not is an
unfinished step, however done it looks. **Before reporting any step or the work as "done", run the
sync check:** plan, `activeContext.md`, and the code must agree; if they diverge the code is the
truth, so reconcile the docs first, then report.

### Fast path

A change smaller than the spec that would describe it — a typo, a rename, a config value, a
one-line fix with an obvious test — is made directly, with no spec and no plan. Say that you
are taking the fast path. A fast-path fix with regression risk requires a test in the same
change set — no test, no fast path — and a note in `.memory-bank/activeContext.md`.
The ceremony is the method's servant, not its point.

## Style & Output Preferences (MUST MAINTAIN)

### Rule: Preference capture (high priority)

When the user, in any language, states a coding style or output preference, asks for a
rewrite ("more idiomatic", "without LINQ"), or asks to remember a lasting do or don't
("no comments from now on", "remember this"): acknowledge briefly, **immediately** record it
as a bullet below — replacing any bullet it contradicts, and saying so — and follow it
strictly from then on. Bullets here load in every session; that is what makes them binding.
This section is never finished.

### Current preferences

- **Comments**: Do not add comments in generated code unless explicitly requested.
- **Formatting**: Follow the project's formatter / linter configuration when present.

## Architecture & Design Snapshot (MUST SYNC)

Agents orient by this snapshot — find modules and entrypoints here instead of scanning the
tree — so it is only useful while it is true.

### Rule: Run the architecture update unprompted

When a change adds, moves or deletes **source** modules, entrypoints or top-level folders —
not specs, plans or Memory Bank files — or the workspace no longer matches this snapshot:
run the `/sdd-architecture-update` workflow yourself, in the same change set, exactly as if
the user had typed it (its body lives in `.claude/commands/sdd-architecture-update.md`).
Typing it manually is the fallback for when this automatic run visibly did not happen.

```yaml
# last reconciled: never
architecture:
  style: 'TBD'
  entrypoints:
    - 'TBD'
  modules: []

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GregorBiswanger/featherspec](https://github.com/GregorBiswanger/featherspec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
