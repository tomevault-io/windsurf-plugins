---
trigger: always_on
description: These rules define the durable architecture, safety, coding, testing, and delivery
---

# Husklet instructions

These rules define the durable architecture, safety, coding, testing, and delivery
standards for Husklet and its integrated C execution engine. Apply them to every new package and improve
nearby code when doing so preserves behavior and remains within scope.

The retired C engine in `../engine` is a read-only behavioral and performance oracle
during migration. Husklet is the active repository and owns the integrated C engine, Rust control plane, containers,
workspaces, terminal, and desktop application. Do not add GPU, graphics translation,
surface, compositor, CUDA, OpenGL, Vulkan, or Wayland implementation back into this
repository. Never edit `../engine` while studying it.

Everything before "Mission" is process rules, and almost every one exists because
it cost real work. If you are about to **measure** anything, or to **count** what
a suite reported, read "Counting and measurement" — it absorbs the seven sections
that used to carry this scattered, among them "Balance the arm order", "A control
that merely seems unaffected is not a control", "`bench --results` is a resumable
ledger", "Identical source does not mean an identical binary" and "Reading a
profile". "A C change reaching the binary is a separate claim, and it has its own
check" stayed separate and belongs beside them. If you are about to **commit**,
read "What green means" and its subsections. Everything from "Mission" onward is
durable architecture and changes rarely.

**A counter governed by the policy cannot decide the policy.** A lane proposed
replacing a flip budget with a threshold on `direct_declined`, and the counters
appeared to confirm it: the plain guest finished at 1 declined site at every
budget, sqlite at 4 to 26. But `malloc` **alone** declines 17 sites on the
plain guest. It reads 1 in the whole sequence only because the budget's hold
has already fired and suppressed direct mode — and suppressed direct mode
cannot guard-fault, which is the sole entrance to the set. The evidence for
the replacement was produced by the thing being replaced.

Implementing it settled what the counter could not: `admitted` fell 255,303 to
1,228, the sqlite phase regressed 1.74x, and both guests latched inside
`malloc` and lost direct authority for the rest of the process. Before keying
a decision on a counter, ask what writes to it and whether the current policy
gates that write.

## Reading code: CodeGraph first

This repository is indexed by CodeGraph (`.codegraph/` at the root). Reach for it
**before** grep, find, or opening files, both to answer a question and before
editing a symbol. One `codegraph_explore` call returns the verbatim,
line-numbered source of the matching symbols grouped by file — safe to edit from,
and equivalent to having read them — plus the call path among them and a blast
radius naming every caller and the tests that cover each symbol. Prefer the MCP
tool `codegraph_explore`; `codegraph explore "<names>"` in a shell prints the same
output when the tool is unavailable.

The blast radius reports **`no covering tests found`** per symbol. Treat that as a
first-class signal: it names the places where a green suite proves nothing, which
is where this codebase has repeatedly hidden defects.

Two failure modes, both observed:

- **Query precise symbol names, two to four at a time. Never bare filenames.**
  A filename matches repo-wide — `pool.rs` pulls in unrelated container and
  launcher files and spends the whole budget on them.
- **Output is budget-truncated and truncation is silent.** A broad query can drop
  the symbol you asked about and leave it visible only in the blast radius. If the
  source you needed is not in the reply, ask again with fewer, narrower names
  rather than assuming it does not exist.

Do not re-open a file whose source CodeGraph already returned.

`no covering tests found` is a lead, not a verdict. It has misfired repeatedly,
including on symbols whose removal reddens both unit and integration tests.
Confirm a gap by mutating the symbol and watching the suite, which is the standard
this repository requires for a coverage claim anyway.

## What "green" means

The corpus is not the gate. The `nix flake check` verification derivation runs
`cargo test --workspace --all-targets`, which covers the Rust library tests and
the native-package integration tests; the corpus reaches neither.
Eleven stale assertions once survived because every lane ran the corpus and a
targeted `cargo test -p <pkg>` and nobody ran the workspace.

Before committing, run `cargo test --workspace --lib --bins` and
`cargo test -p hl-native --all-targets` — about a minute, and it catches that
whole class. Run them in debug or inside `nix flake check`: under `--release`,
`hl-log`'s verbose tests compile out and the daemon tests need
`HL_ALPINE_ARCHIVE`, so both report spurious failures.

### Two green branches can merge to a red tip

`cargo test --workspace --lib --bins` on each branch proves nothing about the merge.
One lane turned `Signal` from a seven-variant enum into a `1..=64` value type
while another added a test spelling `Signal::Kill`; both were green alone and
the tip did not compile. Git reported no conflict, because there was none —
the conflict was semantic.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [husklet/husklet](https://github.com/husklet/husklet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
