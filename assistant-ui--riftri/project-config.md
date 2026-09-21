---
trigger: always_on
description: This file is the durable starting context for coding agents working on Riftri.
---

# Riftri agent guide

This file is the durable starting context for coding agents working on Riftri.
Read `PROJECT.md`, `ROADMAP.md`, and `docs/architecture.md` before changing the
product boundary or implementing a new storage backend.

## Mission

Riftri is an opt-in copy-on-write storage accelerator for real Git linked
worktrees.

Git remains the source of truth. Users and agents continue to use ordinary Git
commands. Riftri changes only how worktree files are materialized and stored.

## Current stage

The repository has completed Milestones 1 through 5: the capability/Git-semantics
foundation, explicit APFS prototype, recoverable storage lifecycle,
process-scoped transparent Git compatibility, and Linux native backends. On
macOS, supported Linux volumes, and Windows ReFS, Riftri creates real linked
worktrees from strict native APFS clones, Linux reflinks, or ReFS block clones,
reuses exact-tree immutable bases, persists atomic add journals, rolls back
failures, and recovers interrupted
adds without deleting a changed view. Repository-local activation, the
process-scoped Git shim, and explicitly evaluated sh/bash/zsh or PowerShell
hooks can route supported adds through the same
transaction. Shell status and explicitly evaluated deactivation keep global
per-user hook setup visible and reversible without editing shell profiles.
Process-scoped commands can optionally start from a validated Git worktree root.
Clean and explicitly forced managed removals use a separate recoverable journal;
force intent includes an exact content snapshot, recovery preserves later changes,
and status reports retained-base references and disk usage with repository-aware
repair for incomplete journals. Explicit garbage collection uses its own
recoverable journal and revalidates references under the immutable-base lock.
Status reports unexplained or inconsistent state paths but never deletes them.
Managed move and prune now use separate recoverable journals. Pristine
native-COW views can also be compacted explicitly through a separate recoverable
swap journal. Compaction preserves Git registration and HEAD, rejects tracked,
untracked, and ignored entries, and protects both bases until the active add
journal is updated; OverlayFS compaction remains open. Linux creation
actively verifies `FICLONE` with unnamed temporary files and supports Btrfs and
reflink-enabled XFS without a byte-copy fallback. When reflinks are unsupported,
Linux can select OverlayFS only after an artifact-clean active probe succeeds in
the caller's current mount namespace, either directly or through the explicit
root-owned helper for ordinary unprivileged shells. The helper accepts only
caller-owned mount layouts and mount, identity-checked unmount, or disposable
work reset operations; probe setup stays unprivileged and repository-local
enablement remains separate. Ordinary-user helper views restore checkout
permissions through metadata-only copy-up using non-forgeable
`trusted.overlay.*` metadata; rootless namespaces retain the metacopy-disabled
`user.overlay.*` path. The selected profile persists for repair. OverlayFS add and
clean removal use the same durable transaction, exact private-layer ownership,
mount identity, and token-bound crash-gap recovery. Explicit repair remounts an
active view after a boot change while preserving its private upper layer;
mounted moves remain fail-closed. Windows
creation actively verifies ReFS block cloning and private-write isolation before
mutation and uses the same journaled lifecycle. Riftri still has no forced move
lifecycle path, automatic orphan-state repair, ordinary-NTFS
backend, managed-environment integration, or daemon.
The native COW checkout path accepts an allowlisted deterministic subset of
in-tree attributes (`text`, `eol`, and `binary` semantics) plus canonical Git
LFS paths backed by strict v1 pointers and verified objects already present in
the default local LFS store. External attributes, custom LFS storage or pointer
extensions, custom filters, encodings, ident substitution, legacy attributes,
and unknown attribute names remain fail-closed.

Milestone 4's transparent-Git compatibility matrix lives in
`crates/riftri-cli/tests/global_activation.rs`. Global per-user shell activation
never replaces repository-local consent; preserve that distinction in UX and
tests.

Check `ROADMAP.md` before starting implementation. Do not skip milestone safety
or compatibility gates merely to reach a working demo faster.

## Hard product boundaries

Riftri does not own:

- Repository cloning or fetching.
- Branch, checkout, commit, merge, rebase, push, or pull semantics.
- Pull requests or Git hosting behavior.
- An agent-specific filesystem API.
- A replacement workspace abstraction.

Riftri owns:

- Explicit or opt-in interception of Git worktree lifecycle operations.
- Preparation and reuse of immutable bases for exact Git trees.
- Native copy-on-write views and their private changes.
- Backend capability detection, cleanup, recovery, and disk accounting.

## Required user experience

The explicit interface is the correctness baseline:

```console
$ riftri worktree add ../app-auth -b feature/auth main
```

The preferred agent interface is process-scoped activation:

```console

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [assistant-ui/riftri](https://github.com/assistant-ui/riftri) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
