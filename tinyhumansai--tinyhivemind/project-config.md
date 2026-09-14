---
trigger: always_on
description: This file is the single source of truth for how humans and coding agents work
---

# Repository Guidelines

This file is the single source of truth for how humans and coding agents work
in this repository. `CLAUDE.md` is a symlink to this file, so every agent reads
the same instructions.

## Charter

`tinyhivemind` is **hive mind mechanics for agents**: a shared session
transcript that several agents read and write, the mechanism by which a message
triggers the right agent to run a turn, and the swarm mechanisms by which a
room of them reaches a decision. Stigmergy, decaying salience, quorum sensing,
cross-inhibition and response thresholds, all as pure folds.

It answers four questions and holds no state doing it — who is here, what a desk
is and who is on it, who an authored mention addresses, and what one participant
sees of the shared transcript.

Three rules decide what belongs here:

1. **The host owns storage.** This repository never opens a database, a file, or
   a socket. `crates/tinyhivemind-core` is a pure algebra; `crates/tinyhivemind` owns
   *ports* a host implements, and nothing more. In particular there is no second
   append-only journal: messages are addressed by sequence number across
   surfaces the host owns, so a second log could not be made consistent with the
   first.
2. **No host types, ever.** Nothing here may name a type from a consuming
   application. A snapshot or a borrowed view crosses the boundary, never a
   callback into the host — a callback seam is how the layering violation this
   crate exists to fix grew in the first place.
3. **One message, one round, of bounded width.** A step may authorize several
   turns to run concurrently — seats are async sessions and the algebra says so
   — but never more than `round_width`, and never without an approval in sight.
   The bound is the invariant; the serialization never was. See
   [ADR 0014](docs/adr/0014-a-round-authorizes-concurrent-turns.md), which
   supersedes ADR 0002 on the terms ADR 0002 itself set.

`crates/tinyhivemind-core` additionally may not depend on an async runtime, a
transport, an HTTP client, a web framework, a SQL database client, a git
implementation, or `anyhow` (non-exhaustive — see the enumerated list in
`.github/scripts/assert-pure.sh`, the source of truth): it is linked into the
hot path of every agent turn and must compile in a host's default build with no
feature flags behind it. `.github/scripts/assert-pure.sh` asserts this, and it
is not advisory — do not add an exception to it to land a change.

`ROADMAP.md` holds the phase plan and the two defects this work exists to fix.

## Project Structure

This is a Rust 2024 cargo workspace rooted at a virtual `Cargo.toml`. Every
crate lives under `crates/`, one directory per package, each directory named for
the package it holds. There is no root package.

```text
Cargo.toml              # virtual workspace: members, [workspace.package],
                        # [workspace.dependencies], [workspace.lints]
crates/
├── tinyhivemind-core/     # the pure algebra: no async, no IO, no host types
│   └── src/
│       ├── lib.rs      # crate docs + the entire public re-export surface
│       ├── error/mod.rs      # crate-wide `Error` and `Result<T>`
│       └── <feature>/        # one directory per feature area
│           ├── README.md     # the module's design, surface, and constraints
│           ├── mod.rs        # module docs, wiring, smallest useful public API
│           ├── types.rs      # substantial type definitions
│           └── test.rs       # module-local unit tests, or a test/ directory
│                             # of behavior-grouped submodules once it grows
├── tinyhivemind/          # the session runtime: ports, the paging walk, the
│                       # responder ladder. Lands in P4; see ROADMAP.md.
└── tinyhivemind-hive/     # bounded group deliberation: traces, salience, quorum
                        # with cross-inhibition, the attention market, and the
                        # episode state machine. Pure, opt-in; lands in P8.
docs/
├── specs/              # behavior and architecture specifications
├── plans/              # test-first implementation plans
├── adr/                # immutable architecture decision records
├── research/           # the reading behind a mechanism, with its equations
└── experiments/        # what happened when it was actually run
wiki/                   # the GitHub wiki, checked out as a submodule
```

### The two-crate split

`crates/tinyhivemind-core` holds the algebra: desks and membership, the roster, the
mention grammar and its resolution, and the fold that projects a shared
transcript into one viewer's turn history. Every function there is a fold over
data the caller already holds. P4 (see `ROADMAP.md`) replaces the
`(role, content)` pair this fold produces with an attributed `SessionMessage`
in `crates/tinyhivemind` — the projection *algorithm* stays here in core, but the
richer, attributed shape it produces is assembled by the crate that also owns
the paging walk over a live session log.

`crates/tinyhivemind` holds the parts that must wait on something — the paging walk
over a session log, the responder ladder, the mention-dispatch edge — expressed
against ports a host implements. It depends on the core crate and re-exports it,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tinyhumansai/tinyhivemind](https://github.com/tinyhumansai/tinyhivemind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
