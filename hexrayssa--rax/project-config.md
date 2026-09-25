---
trigger: always_on
description: This file is the root execution contract for autonomous and interactive coding
---

# RAX Agent Engineering Guide

## 1. Purpose and scope

This file is the root execution contract for autonomous and interactive coding
agents working in this repository. It applies to the entire tree unless a deeper
`AGENTS.md` supplies more specific instructions. It is optimized for changes
whose correctness depends on exact CPU, memory, exception, floating-point,
vector, ABI, or JIT behavior.

RAX is a Rust 2024 multi-ISA emulator and virtual-machine monitor. Its central
correctness problem is not only implementing an instruction once; the same
architectural behavior may be represented in the direct ISA interpreter, SMIR
lifter and interpreter, native lowerers, backend adapters, static analysis API,
and differential tests. An agent must identify every affected representation.

Do not duplicate the README's volatile instruction-coverage claims here. For
current advertised capabilities, consult `README.md`; for implementation truth,
inspect source and executable tests.

## 2. Instruction priority and truth hierarchy

Apply instructions in this order:

1. System, developer, and current user instructions.
2. The nearest applicable `AGENTS.md`.
3. Repository configuration and executable behavior.
4. Current source and tests.
5. Current CI workflow definitions.
6. Derived design documents and historical reports.
7. Comments, issue prose, and names.

When sources disagree, do not silently select the convenient one. Record the
conflict, determine which source is authoritative for the task, and update stale
documentation in scope. If the answer remains unknown, state `unknown`, list
the assumption required to proceed, and provide a falsification probe.

Repository-specific examples:

- `Cargo.toml` is authoritative for feature names and Cargo test targets.
- `.github/workflows/*.yml` is authoritative for current CI commands.
- `src/lib.rs` and `src/README.md` are authoritative for canonical module
  ownership and compatibility re-exports.
- Rust source is authoritative over the dated SMIR markdown baseline in
  `docs/specifications/smir/`.
- ISA manuals define architectural behavior; tests and code must not redefine
  specified behavior merely to agree with each other.
- A successful command is not evidence that an oracle or host-specific test ran;
  inspect test counts and skip output.

## 3. Universal operating contract

### 3.1 Technical rigor

- Do not fabricate paths, APIs, feature support, test results, references, or
  architectural semantics.
- Separate facts, inferences, assumptions, and unknowns.
- Use exact architecture nomenclature and fixed-width types. Express sizes in
  bytes or bits explicitly. Use SI units for time/rate and hexadecimal for
  addresses, masks, encodings, and architectural bit fields where it improves
  auditability.
- Show calculations stepwise when a result depends on widths, masks, address
  arithmetic, scaling, timing, or layout. Include units, truncation rules,
  overflow behavior, significant figures, and error bounds where applicable.
- For algorithms, state relevant time and space complexity when it informs the
  implementation or review.
- Make technical tradeoffs from evidence. Do not provide ethical opinions. If a
  request cannot be completed without an ethical judgment, output `ETHOUT`.

### 3.2 Assumption Register

Maintain an Assumption Register for every nontrivial task. Each assumption must
have:

| Field | Required content |
|---|---|
| ID | Stable identifier such as `A1` |
| Assumption | Precisely what is being treated as true |
| Basis | Source evidence or reason it is necessary |
| Dependent result | Which design, edit, or conclusion depends on it |
| Stress test | Boundary or adversarial case |
| Falsification probe | Concrete observation or command that would disprove it |
| Status | confirmed, retained, revised, or falsified |

Do not invent assumptions to populate the table. If none materially affect the
result, report `None`. Revisit the register after implementation and testing.

### 3.3 Bounded scope

Complete the requested behavior rather than the smallest change that happens to
make one test pass. At the same time, do not mutate adjacent systems without
authorization.

Record discovered out-of-scope items separately:

| Impact | Meaning |
|---|---|
| High | Can invalidate correctness, safety, ABI, or the requested result |
| Medium | Material maintainability, performance, portability, or coverage issue |
| Low | Local cleanup or optional improvement |

For each item, cite evidence and state whether it blocks the task. Do not
implement a non-blocking opportunity merely because it was discovered.

### 3.4 Worktree ownership

Assume the worktree may be shared with humans or other agents.

Before editing:

1. Run `git status --short --branch`.
2. Resolve the repository root with `git rev-parse --show-toplevel`.
3. Inspect diffs for every tracked file already modified in the intended scope.
4. Treat all pre-existing tracked and untracked content as user-owned.
5. Record the baseline HEAD and the files this task will own.

During work:

- Re-check status before broad formatting, generation, testing that writes
  fixtures, or any Git operation.
- Never use `git reset --hard`, `git checkout -- <path>`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HexRaysSA/rax](https://github.com/HexRaysSA/rax) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
