---
trigger: always_on
description: Validates convergence, idempotency, cycle handling, coverage.
---

# CLAUDE.md — outl

Context for Claude Code sessions working on this repo. Read this before making any change.

## What this project is

**outl** is a local-first outliner (Roam/Logseq replacement) with:

- **Markdown as source of truth** — `.md` files are 100% clean, no visible IDs.
- **Conflict-free sync** via a tree CRDT (Kleppmann et al. 2022).
- **Trait-based storage** — JSONL (one file per actor) is the only
  persistent backend; ChronDB on the roadmap.
- **TUI as a first-class citizen**, not an afterthought.
- **Journal-first** — daily notes are the primary entry point.

Full spec lives in the README and `docs/`. Don't skim — read.

## Critical invariants (NEVER violate)

These are the non-negotiables. Violating any one breaks user trust irreversibly.

1. **Op log is source of truth.** All mutations go through `Op` → `apply_op` → log.
   The materialized tree and `.md` files are projections. Never edit `.md` to "fix" state.

2. **Markdown stays 100% clean.** No `id::`, no UUID inline, no HTML comments, nothing.
   IDs live ONLY in the `.outl` sidecar (JSON file next to the `.md`, e.g. `pages/foo.outl`).
   The sidecar is **not** a dotfile — iCloud Documents drops dotted paths during cross-device
   sync, which silently breaks multi-device workspaces. Same rule applies to `ops/`.

3. **CRDT follows Kleppmann 2022 literally.** `do_op` / `undo_op` / `apply_op` /
   `creates_cycle` must match the paper. 100% coverage on these four is non-negotiable.

4. **Move that creates a cycle is a no-op on the materialized tree, but the op
   still goes into the log.** Removing it breaks correctness of future reordering.

5. **Storage is a trait, not a struct.** `JsonlStorage` is the only
   persistent impl; tests use `MemoryStorage`. Anything that wants
   to persist ops goes through `dyn Storage`. No second persistent
   backend lands without an issue + RFC first — divergence between
   storages is exactly what we paid to remove in 0.5.0.

6. **Delete is `Move(node, TRASH_ROOT)`, not physical removal.** Simplifies the
   algorithm and preserves history.

7. **Any state that must converge between devices goes through the op log.**
   If two users (or one user on two devices) can disagree about a value
   and you want them to reconcile, the state belongs in an `Op` — *never*
   in a shared file with last-write-wins semantics. The op log gives each
   actor its own `ops-<actor>.jsonl`, lets iCloud / Syncthing / shared FS
   sync per-file (no merge conflicts), and replays through the CRDT with
   HLC ordering for deterministic convergence. Writing the state into the
   sidecar (or any single shared file) bypasses all of that and loses
   concurrent writes silently. **Default position: model it as an Op.**
   `Op::SetCollapsed` for the fold flag is the canonical example. The
   sidecar carries only **structural matching metadata** (ids, position,
   content hash, ref handle) — it is not a sync surface.

## Repo layout

```
outl/
├── CLAUDE.md                  # this file
├── README.md
├── LICENSE                    # MIT
├── Cargo.toml                 # workspace
├── rust-toolchain.toml
├── .claude/                   # agents, commands, hooks, settings
├── .github/workflows/
├── docs/
│   ├── architecture.md        # design decisions
│   ├── crdt.md                # CRDT algorithm details — read this
│   ├── markdown-format.md     # outl dialect + sidecar spec
│   ├── storage.md             # trait Storage + roadmap
│   └── roadmap.md             # 6-phase plan
└── crates/
    ├── outl-core/             # tree CRDT, op log, storage trait
    ├── outl-md/               # parser, sidecar, matching
    ├── outl-actions/          # UI-agnostic workspace ops (shared by every client)
    ├── outl-exec/             # code-block runtime (desktop)
    ├── outl-cli/              # `outl` binary
    ├── outl-tui/              # `outl-tui` binary
    └── outl-mobile/           # Tauri 2 mobile app (iOS first)
```

## Shared logic: `outl-actions`

Every workspace mutation a client needs to perform (edit a block,
toggle TODO, indent / outdent, delete, render today's `.md`) lives in
**`outl-actions`**, not in the client crate. The mobile app and the
TUI must call the **same** functions for the same semantics; if a new
operation needs more than one client, it goes in `outl-actions`
before its first use.

The contract is short:

- Functions take `&mut Workspace` and `&HlcGenerator`.
- They route every mutation through `Workspace::apply` (op log
  stays source of truth).
- They never hold UI state and never touch storage backends directly.

See `crates/outl-actions/CLAUDE.md` for the full surface and the
"what this crate does NOT own" list. **If you find yourself writing
tree-walking or op-building helpers inside `outl-tui/`,
`outl-mobile/`, or any future client, stop and put them in
`outl-actions` first.** The TUI's `outline_ops.rs` is the one
deliberate exception (it manipulates an in-flight AST that hasn't
been parsed back to a workspace yet — see that file's module doc).

Per-crate context lives in `crates/<name>/CLAUDE.md`. Read it before editing
that crate.

User-facing docs in `docs/`:

- `docs/crdt.md` — the algorithm and its invariants.
- `docs/architecture.md` — design decisions.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [avelino/outl](https://github.com/avelino/outl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
