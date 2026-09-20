---
trigger: always_on
description: Instructions for coding agents (Claude, Codex, Grok, Cursor, and others)
---

# Agent notes — jevify

Instructions for coding agents (Claude, Codex, Grok, Cursor, and others)
working in this repository. These rules are **mandatory** unless the user
explicitly overrides them for a task.

This file governs **how** work is done here. **What** the project is lives in
[`docs/00-invariants.md`](docs/00-invariants.md): purpose, vocabulary, the
numbered invariants, the design criteria and the score-semantics contract.
That document ranks above every ADR and above this file. Read it before
changing anything. Decisions that constrain future work are ADRs under
`docs/adr/`, indexed in [`docs/adr/README.md`](docs/adr/README.md).

Four principles bind every change, restated here for a project that started
empty: Always Works, test-driven development, SOLID, and one authoritative
path per process.

## A. Always Works™ — proof before done

**"Should work" ≠ "does work."** Before marking any change complete, prove
it with evidence you personally observed in this session, not with
assumptions, not with "the build succeeded", and not with a result from an
earlier state of the tree.

| Change type | Minimum proof |
|---|---|
| Code | The test suite runs green against the **working tree**, in the interpreter and dependency set the project pins. |
| Anything that runs, serves, loads or scores | Run it and observe its output. A passing unit test is not a running program. |
| A number in a document | Comes from a run whose command, inputs and their hashes, model revisions and software versions are recorded next to it. Re-derive; do not copy from memory. |
| Docs only | No runtime required; still re-read for accuracy against the tree. |
| Config, CI, scripts | Execute the path that changed. |

**The stale-artifact trap.** Anything cached is evidence about the past:
a built image, a downloaded checkpoint, a feature cache, a saved run, a
result file. Before a cached thing counts as proof, show that it
corresponds to the working tree (rebuild it, or compare a recorded hash or
revision). A green result on a stale artifact is a silent false green.

**Name what is unproven.** When done, state plainly which paths were
exercised and which were not. A claim of completion that hides an untested
path is a defect in the claim, not in the code.

## B. Test-driven development — new behavior is test-first

Do not write production code for a new feature, a bug fix with a known
reproduction, or a new module until a failing test names the behavior.

| Rule | Detail |
|---|---|
| **Red → green → refactor** | Write one failing test; write the minimum code that passes; only then improve structure. |
| **Vertical slices** | One seam, one behavior at a time. Do not bulk-write a suite of imagined tests and then implement everything. |
| **Agree the seam first** | Before the first test, state the public interface under test: a function, a Protocol, an HTTP path, a CLI command. Tests hit that seam only. |
| **No private tests** | Do not assert on private helpers, internal call counts or implementation structure. Prefer fakes at port seams. |
| **Independent expected values** | Assertions use known literals or domain rules, never a recomputation of the algorithm under test. |
| **Where tests live** | `tests/`, runnable from a clean checkout with one documented command. |
| **When TDD does not apply** | Pure renames, docs, configuration, mechanical refactors with no behavior change. Existing tests still run (principle A). |

Measured results are tests too. An experiment has a protocol written and
hashed **before** the run, a fixed evaluation set, and no post-hoc tuning
against the test split. A result that was selected on the test set is not
a result.

## C. SOLID — deep modules behind narrow seams

Prefer **deep modules** (small interface, large behavior) over shallow
pass-through wrappers. Vocabulary for structure: *module, interface,
implementation, depth, seam, adapter, port, composition root.*

| Principle | In this repository |
|---|---|
| **S** — Single responsibility | One reason to change per module. A module that mixes core logic with a vendor client, a model runtime or a transport has two. |
| **O** — Open/closed | Extend by adding an adapter or a new caller of a deep module. Do not fork a spine and edit the copy. |
| **L** — Liskov | Every adapter of a port is substitutable, including test fakes. A fake that honors less of the Protocol than production is a lying test. |
| **I** — Interface segregation | Focused Protocols. No object that callers use ten percent of. |
| **D** — Dependency inversion | Core logic depends on **ports** (Protocols), never on concrete adapters, model libraries, HTTP clients or files. One composition root wires adapters into the core; nothing else constructs infrastructure. |

**The deletion test.** If deleting a module only moves lines around and no
complexity concentrates anywhere, it was shallow. Do not add more of those.

Anti-patterns to reject on sight:

- Implementing first and "adding tests later".
- Core logic typed against a concrete runtime, client or store instead of a port.
- A blanket `except Exception` that swallows a failure without logging it and naming its contract.
- Weakening an invariant in production code to make a test pass.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fidecastro/jevify](https://github.com/fidecastro/jevify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
