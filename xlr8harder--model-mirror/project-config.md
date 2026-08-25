---
trigger: always_on
description: This file applies to the entire repository. It is the high-level contract for
---

# Agent Guide

This file applies to the entire repository. It is the high-level contract for
agents and contributors working on model-mirror. Detailed development guidance
and commands live in [CONTRIBUTING.md](CONTRIBUTING.md).

## Project Contract

Model-mirror exists to preserve usable, independently verifiable copies of
model repositories when upstream changes, removes files, becomes unavailable,
or disappears.

Preservation takes priority over convenience:

- Resolve mutable upstream revisions to immutable commits before changing an
  archive.
- Treat repository type, repository ID, and resolved commit as snapshot
  identity. Never silently mix files from different commits.
- `repair` restores the recorded commit. Moving to another commit is an
  explicit update that users can preview before applying.
- An upstream change or failure must not silently rewrite, invalidate, or
  delete a sound local archive.
- Retain enough local evidence to inspect, verify, repair, and distribute a
  snapshot without assuming upstream will remain available.

Model payload formats are opaque bytes. Do not make model-mirror a validator
for safetensors, GGUF, tokenizer, or framework-specific semantics unless a
separate, actionable feature is deliberately added.

## Safety And Trust

Use a defensive posture throughout the project:

- Treat upstream metadata, torrent metadata, downloaded paths, cache records,
  and existing filesystem state as untrusted input.
- Reject traversal, absolute paths, duplicate canonical paths, reserved-path
  collisions, symlink escapes, and non-regular payload files.
- Resolve exact targets before mutation or deletion. Never broaden a cleanup
  from malformed or inferred data.
- Prefer atomic same-filesystem finalization and atomic metadata replacement.
  Interrupted work should be detectable, resumable, or safely removable.
- Preserve unrelated local files unless the user explicitly selected them for
  removal.
- Keep completeness, local integrity, upstream provenance, publication trust,
  and upstream availability as separate claims.

Torrent publications are immutable and belong to one resolved commit. Torrent
piece verification proves agreement with the torrent metadata, not authentic
upstream provenance. Active publications fence commit-changing updates until
they are explicitly retired. Standard torrent clients must remain a supported
escape hatch, and non-torrent workflows must not require libtorrent.

Large archives commonly live on HDDs:

- Avoid unnecessary payload rereads, random I/O, duplicate full snapshots, and
  unbounded concurrency.
- Accumulate required hashes during an existing byte pass where practical.
- Commands presented as fast metadata inspection must not secretly scan or
  hash payloads, contact upstream, or mutate state.

## Ways Of Working

Inspect the current source, tests, and state formats before relying on older
notes or handoffs. Preserve unrelated changes in a dirty worktree and keep each
change scoped to the requested behavior.

Design behavior so it can be tested without live services:

- Keep comparison, planning, validation, and state-transition logic
  deterministic where practical.
- Put network access, filesystem mutation, clocks, processes, and torrent
  backends behind explicit boundaries.
- Make advisory and dry-run operations genuinely non-mutating.
- Consider failure, interruption, restart, and stale-state behavior when
  changing a workflow.

The repository requires 100% statement and branch coverage. Preserve the gate;
do not lower it or add exclusions to land a change. Start behavior changes with
the smallest tests that demonstrate the observable contract, then implement
and run the full coverage suite.

Coverage is a backstop, not a request to turn every sentence into a test.
Prefer tests for behavior, invariants, compatibility, and defensive refusals.
Do not freeze incidental wording, implementation details, or exploratory
design notes unless they are a real contract.

## CLI Contract

The CLI is the product interface.

- Default output is readable, formatted, quickly scannable, and centered on
  state, impact, and the next useful action.
- Summary views emphasize exceptional states; repository detail views show
  useful identity, size, file, commit, age, and operational information.
- Failures explain categorized causes and give exact next commands when a safe
  next action exists.
- Potentially large output is summarized by default with a discoverable way to
  show all of it.
- Add `--json` where inspection, planning, or reporting has a concrete
  automation use case. JSON is a stable, versioned schema, not a dump of
  internal objects.
- Keep JSON stdout free of prose and progress. Send progress to stderr.
- Expensive work, network access, and mutation must be explicit in command
  names, flags, help, and documentation.
- Human formatting must not depend on color, and important semantics must be
  consistent across output, JSON, help, and state files.

## Documentation And Change Discipline

Keep the README a concise front door. Put detailed user workflows in the
focused guides under `docs/`, implementation and testing mechanics in
`CONTRIBUTING.md`, release mechanics in `RELEASING.md`, and user-visible changes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xlr8harder/model-mirror](https://github.com/xlr8harder/model-mirror) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
