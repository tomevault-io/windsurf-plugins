---
trigger: always_on
description: This document is for coding agents working on the Workboard codebase.
---

# AGENTS.md

This document is for coding agents working on the Workboard codebase.

## Project operations

- Do not create Linear issues, stories, or subissues for Workboard unless a future task explicitly asks for them.
- If implementation work changes code, open a GitHub PR after verification.
- If Workboard behavior changes, update the bundled `using-workboard` skill in `skills/using-workboard/` in the same change so agent guidance stays current. Treat `skills/using-workboard/SKILL.md` and `skills/using-workboard/references/current-cli.md` as required touchpoints whenever CLI behavior, file protocol expectations, or supported workflows change.

## Mission

Build a **Go-based**, **cross-platform**, **file-native ticket management system** for software teams and coding agents.

The product is optimized for **agents first** and **humans second**.

The filesystem protocol matters more than UI polish. Preserve the protocol, the invariants, and recovery properties.

## Non-negotiable constraints

1. **Filesystem is the source of truth** in v1.
2. **No required database**.
3. **Cross-platform support is required** for macOS, Windows, and Linux.
4. **Synced-folder safety matters more than real-time behavior**.
5. **Use append-only event files** for shared activity.
6. **Treat file watching as optional optimization**, never a correctness dependency.
7. **Assume consumer sync latency and occasional conflict copies**.
8. **One active implementation claim per ticket should win**.
9. **Queues and indexes are derived state**, never canonical state.
10. **Git remains the system of record for source code changes**.

## Product summary

Workboard is a filesystem protocol plus a CLI/reconciler.

It should allow multiple coding agents to coordinate work by reading and writing plain files inside a shared folder.

The implementation must be easy to inspect, easy to script, and easy to recover after partial corruption or sync weirdness.

## Build priorities

Priority order:

1. stable file protocol
2. validation and recovery
3. claim semantics
4. deterministic indexing
5. CLI usability
6. optional watch mode

Do not optimize for interactivity before the protocol is solid.

## MVP deliverables

Implement these commands first:

- `workboard init`
- `workboard new`
- `workboard show`
- `workboard claim`
- `workboard release`
- `workboard event`
- `workboard index`
- `workboard validate`
- `workboard doctor`
- `workboard reconcile`

### Command expectations

#### `workboard init`
Create a fresh folder scaffold with templates and base config.

#### `workboard new`
Create a new ticket folder with `ticket.md`, `spec.md`, `plan.md`, `result.md`, plus `events/` and `claims/` directories.

#### `workboard show`
Display a ticket summary by reading canonical files.

#### `workboard claim`
Create or replace the current actor's claim file for a ticket. Emit a `claim-acquired` event.

#### `workboard release`
Mark or remove the current actor's claim and emit `claim-released`.

#### `workboard event`
Append a well-formed event file to a ticket.

#### `workboard index`
Rebuild derived indexes and queue files from canonical state.

#### `workboard validate`
Parse and validate canonical files without mutating them.

#### `workboard doctor`
Detect broken invariants and report repair guidance.

#### `workboard reconcile`
Resolve derived state and lightweight conflicts. It may also expire claims and surface contested tickets.

## File protocol invariants

Preserve these invariants.

### Invariant 1: Canonical state is local to the ticket or project root
Canonical files:
- `project.yaml`
- `agents/*.yaml`
- `tickets/*/ticket.md`
- `tickets/*/spec.md`
- `tickets/*/plan.md`
- `tickets/*/result.md`
- `tickets/*/events/*.yaml`
- `tickets/*/claims/*.yaml`

### Invariant 2: Derived state must be safe to delete
Derived files:
- `indexes/*.json`
- `queues/*.txt`

### Invariant 3: Shared activity should prefer new files
When multiple actors may act concurrently, prefer creating a new event file rather than editing a shared file.

### Invariant 4: Claim conflicts must be detectable and resolvable
If multiple active implementation claims exist, the reconciler must determine the winner deterministically and flag the ticket as contested.

### Invariant 5: Ticket summary files stay compact
`ticket.md` should remain easy for an agent to read quickly.

## Deterministic conflict policy

When two active implementation claims exist for the same ticket:
- earliest unexpired claim wins
- tie-break by lexical `actor_id`
- both claims remain visible
- reconciler records the contest in derived state or emits a warning
- no data is silently discarded

## Suggested repository layout

```text
/workboard-cli/
  cmd/
    workboard/
      main.go
  internal/
    app/
      app.go
    cli/
      init.go
      new.go
      show.go
      claim.go
      release.go
      event.go
      index.go
      validate.go
      doctor.go
      reconcile.go
    model/
      project.go
      agent.go
      ticket.go
      claim.go
      event.go
      index.go
    parser/
      frontmatter.go
      markdown.go
      yaml.go
      time.go
    fsstore/
      workboard.go
      tickets.go
      claims.go
      events.go
      atomic.go
      paths.go
    reconcile/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sidkusnanto/workboard](https://github.com/sidkusnanto/workboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
