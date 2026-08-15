---
trigger: always_on
description: Ravel is a multi-tenant telemetry database. S3-compatible object storage is
---

# Ravel: agent instructions

Ravel is a multi-tenant telemetry database. S3-compatible object storage is
the only durable backend; every compute process is disposable. These rules
apply to every agent working in this repository, including unattended fleet
executors.

## Unattended behavior

- Never ask for confirmation or approval. When your work passes the gates,
  commit it and finish with a report. An unanswered question ends the task
  with the work lost.
- If you find a contradiction between a spec document and code, or a bug in
  a crate outside your task scope, report it in your final message. Do not
  silently fix or work around it.

## Workspace isolation

- Always work in a dedicated git worktree, never directly on the primary
  checkout. Create one (`git worktree add`) before making any change, and
  remove it once your work is merged. This applies to every agent,
  including a local subagent dispatched into this same repo: a subagent
  editing files directly in the dispatching session's working tree, or
  two subagents sharing one tree, corrupts both in-flight edits and any
  concurrent `cargo` build cache. One worktree per unit of work, always.
  This rule has no small-change exception. A doc-only edit and a one-file
  fixup follow it too: both commits that ever bypassed it were rationalized
  as "just a doc" and "just one file", and a concurrent session can sit on
  the primary checkout at any moment.
- Exception: fleet executors working in a dedicated clone. The clone is
  already the isolated workspace; commit directly on the dispatched
  checkout's HEAD (detached HEAD is fine). Do not create a side worktree
  or branch: the fleet harness collects only the dispatched checkout's
  HEAD as the result, and work committed anywhere else is lost when the
  workdir is destroyed (this happened; see the 2026-07-27 audit report,
  section 10).

## Merging fleet results

- A real (non-fast-forward-only) merge conflict between a fleet result and
  current `main` can mean two different things: overlapping edits (resolve
  textually), or a structural decision landed on `main` while the task was
  in flight and the task's whole premise is now stale (an ADR, a format
  version change, a crate rewritten from scratch). Before resolving, read
  the commit(s) on `main` that conflict — `git log --oneline
  <merge-base>..origin/main -- <conflicting paths>`, then the full commit
  body of whatever touched the same files. Forcing a stale-premise branch
  through reintroduces code or assumptions a deliberate decision already
  removed.
- This happened twice on 2026-07-28: ADR-0027 (single-RSEG-version
  pre-release) landed mid-flight under two long-running tickets built on
  the multi-version model it deleted. One had a partial file-level
  collision (some files merged clean, one file conflicted because it had
  already been rewritten for the new reality); the other's whole
  dependency chain (a path dev-dependency on a crate independently
  rewritten from scratch) needed re-targeting, not just conflict
  resolution.
- If the underlying logic (not the version/format-specific plumbing) is
  still valuable once the premise moves, don't discard it and don't force
  it through: preserve the branch, comment on the relevant issue with a
  pointer to it as reference material, and let a follow-up port it onto
  the new reality deliberately.

## Invariants (violating these is never a valid trade-off)

- Object storage is the source of truth. No durability may depend on local
  disk, and no recovery path may read state another process wrote locally.
- Data objects, commit records, manifests, and index objects are immutable.
- Persistent formats are frozen contracts: the RSEG layout
  (docs/segment-format.md), the protobuf schemas under proto/, canonical
  series identity and commit tokens (crates/ravel-types), and the object
  key layout (docs/catalog-and-mvcc.md). Changing any of them requires an
  ADR and a version bump, never an in-place edit.
- `unsafe` is denied workspace-wide. No unwrap/expect in production code
  paths; test modules carry `#[allow(clippy::expect_used)]`.
- Exact semantics by default. Approximation is opt-in and visible.
- No placeholder implementations on critical paths; no TODO that changes
  durability or query correctness.

## Gates (run all before any commit; CI runs the same)

```sh
cargo fmt --all --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test -p <your-crate>        # plus --workspace when your change is cross-crate
```

None of those compile `ravel-server`'s SQL or Flight SQL surfaces: both sit
behind cargo features that are off by default. When your change touches
`ravel-server`, `ravel-sql`, or `ravel-query`, add:

```sh
cargo clippy -p ravel-server --features sql --all-targets -- -D warnings
cargo test   -p ravel-server --features sql
cargo clippy -p ravel-server -p ravel-sql --features flight-sql --all-targets -- -D warnings
cargo test   -p ravel-server -p ravel-sql --features flight-sql
```

`scripts/gates.sh` runs these when the crates are in scope. Do not skip them:
a workspace gate has printed "All gates passed" on a tree where
`--features sql` failed to compile, because the broken call site sat in a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NOFireAI/ravel](https://github.com/NOFireAI/ravel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
