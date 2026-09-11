---
trigger: always_on
description: Public guidance for AI coding agents working on Ironwood. See
---

# AGENTS.md

Public guidance for AI coding agents working on Ironwood. See
[CONTRIBUTING.md](CONTRIBUTING.md) for contribution requirements and the project's
AI-assisted development policy. Keep durable rules here; keep feature catalogs,
history, examples, and specifications in the linked documents.

## Checkout, edits, and Git

For maintainer-directed agent tasks, follow this workflow unless the human
explicitly instructs otherwise. External contributors follow the pull request
process in `CONTRIBUTING.md`.

- Use the task's designated canonical checkout for `ironwood-lang/ironwood`.
  Before reading or changing repository files, verify that the working directory
  is that checkout's root and that `origin` uses
  `https://github.com/ironwood-lang/ironwood.git` for both fetch and push. Stop on
  a mismatch; do not change files, remotes, branches, or history to repair it.
- Work directly on local `main`. Do not create or switch branches or create
  worktrees unless the human explicitly requests it.
- Before editing, fetch `origin` and fast-forward `main` to `origin/main`.
  Report a blocker if this cannot be done safely.
- Treat current file contents as authoritative, including committed and
  uncommitted human edits. Immediately before each edit, re-read the affected
  file or region. Apply the smallest patch; never restore an earlier agent
  version or rewrite unrelated content. Inspect the diff for unintended
  deletions, reversions, or rewrites. If human edits conflict with the task and
  cannot be preserved safely, stop and ask.
- Verify the change, commit only the task's changes to `main`, fetch again,
  safely integrate new `origin/main` commits, and push only `main`. Fetch
  afterward and confirm zero divergence and a clean, synchronized `main`.
  Never commit unrelated human changes.
- Report specific verification, commit, integration, push, or synchronization
  blockers instead of claiming completion.

## Language and compiler invariants

Ironwood is its own statically compiled systems language, not a Java
implementation or transpiler:

> Java designed to replace C++ instead of to run on a virtual machine.

- Use the behavior a Java programmer expects unless it conflicts with
  closed-world native compilation, explicit safe reclamation, or an accepted
  Ironwood decision. Prioritize high-performance Java-shaped applications
  needing native AOT, not the full Java platform. Before adding a broad subsystem
  for a familiar API,
  evaluate a fixed convention or smaller dependency, resolve material scope
  choices with the human, and document accepted conventions.
- Compile ahead of time to native executables or libraries with a closed-world
  final link. Preserve whole-program reachability, specialization,
  devirtualization, and ownership analysis. Do not introduce JVM bytecode, a JVM,
  JIT, runtime class loading, or JVM machinery to mimic an API; use native or
  compile-time mechanisms, a reduced API, or an explicit omission.
- Keep ordinary source Java-shaped: classes, interfaces, references, packages,
  exceptions, generics, arrays, and control flow. Do not expose raw pointers,
  pointer arithmetic, manual vtables, or Rust-style lifetime syntax.
- There is no GC. Ordinary `new` allocations remain until a compiler-proven
  `free` or process termination. Reject `free` when safety cannot be proved;
  never weaken the guarantee or silently reclaim unreachable objects.
- Keep the mandatory runtime small; unreachable standard-library code is removed
  from the closed-world program, allowing a broad library.
- Use UTF-8 `.iron` source, a Java 21 bootstrap compiler, and the pinned LLVM 23
  backend. Preserve the pipeline: compiler-owned typed IR -> LLVM IR ->
  `llvm-as` -> `opt` -> `llc`, plus Clang compilation of the isolated C runtime
  and native linking. Generated programs do not require Java.
- Keep frontend semantics in typed analysis and IR. Do not replace the repository
  skeleton or pipeline, translate Java to C, or implement semantics in ad hoc
  LLVM text generation.

### Safety and performance

- Memory-safety enforcement is mandatory in every compiler mode. Preserve
  compile-time protection against dangling references, use after free, and
  double free. If aliasing, escape, or lifetime facts cannot prove a `free`
  safe, reject it with a compilation error. Never assume unknown effects are
  non-retaining or grant borrowing/ownership exemptions merely to silence
  diagnostics or make code compile.
- Keep missing-free diagnostics (`--unfreed=off|warn|error`) separate from
  memory-safety enforcement; no setting may disable or downgrade mandatory
  safety errors. Fix false positives by correcting analysis or code without
  weakening reclamation proofs. Changes to ownership or escape analysis need
  focused regressions for both accepted safe cases and rejected unsafe cases.
- Safety must not add runtime overhead on valid paths. Prefer compile-time
  proofs and eliminated checks. Do not add scans, hash lookups, registries,
  state tracking, or allocations solely to detect library-contract misuse;
  document caller obligations instead. Constant-time, allocation-free checks
  still cost something. Discuss any safety requirement needing runtime overhead

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ironwood-lang/ironwood](https://github.com/ironwood-lang/ironwood) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
