---
trigger: always_on
description: - We don't care about backwards compatibility because we are still in development. Keep the code simple and lean.
---

# AGENTS.md

## General
- We don't care about backwards compatibility because we are still in development. Keep the code simple and lean.
- Avoid adding new Rust features or ENVs unless it is explicitly approved.
- Never modify this file without explicit approval.
- When creating commits, always include a DCO sign-off (`git commit -s` or an equivalent `Signed-off-by:` trailer).
- No single file should ever exceed 1,500 lines of code unless explicitly confirmed by the user.
- Heavily avoid bloat. We want to maintain a compact and lean codebase.
- Proofs must remain compatible with on-chain verification targets. In proof/transcript/public-digest paths, use Poseidon2-only hashing unless explicitly approved otherwise.
- Do not introduce mixed hash families (e.g., Blake3/SHA prehashes) in protocol-binding paths without explicit user approval.
- For difficult questions, hard design/review tradeoffs, or high-confidence soundness checks, you may use the project-local multi-AI council skill at `./.codex/skills/multi-ai-council/SKILL.md` (it may take between 5 - 25 min to answer).
- You can find the SuperNeo paper which is what the main protocol is based upon in ./docs/superneo-paper

## Design & Architecture
- When evaluating design or architectural decisions, think from first principles: reduce the problem to its irreducible truths—axioms, physical laws, hard constraints—and derive every conclusion strictly from those, rejecting inherited conventions and unstated assumptions.
- Before proposing any architectural change: (1) list every assumption you are making, (2) challenge each by asking "is this a necessity or just a convention?", (3) discard any that fails. Only then derive your answer from what remains.
- Code philosophy north star:
  - John Ousterhout: prefer deep modules with small, stable interfaces and unambiguous ownership.
  - Rich Hickey: prefer simplicity over flexibility theater; do not introduce abstractions, layers, or helper systems until a real repeated need exists.
  - Casey Muratori: prefer explicit data flow, explicit control flow, and mechanically obvious code over cleverness that hides what the machine or proof system is doing.
- Use those principles as a practical test:
  - If ownership is blurry, the design is not done.
  - If a new abstraction mostly moves complexity around instead of removing it, reject it.
  - If understanding a hot path requires chasing wrappers or indirection, simplify it.
  - If a module grows by absorbing unrelated responsibilities, split it by responsibility instead of adding more flags or configuration.
- Rust file/module documentation should optimize for ownership clarity and auditability, not ceremony.
- Do not add top-level file docs to trivial files whose purpose is obvious from the code.
- For normal files, prefer a short `//!` ownership header that states what the file owns and what it does not own.
- For protocol-critical or ABI-critical files, prefer a short contract header that states ownership, inputs/outputs, and invariants.
- Do not use top-level docs for implementation history, migration progress, aspirations, or Jolt/SuperNeo name-dropping without explaining the local ownership boundary.
- Do not write large tutorial-style or paper-recap headers in implementation files; keep top-level docs compact and architectural.

## Testing
- Never add tests in the same implementation file, always prefer to add them to a file inside tests/ (current or new)
- If you add a test to catch a problem, the test should fail if aims to confirm a problem.
- Always use `FoldingMode::Optimized` in tests. Never use `FoldingMode::PaperExact` unless the user explicitly approves it. PaperExact is an O(2^ell) brute-force reference engine meant only for correctness cross-checking, not general test usage.

## Build & Test Commands
- After modifying Rust code, always run `cargo fmt --all` before finishing unless the user explicitly says not to.
- When running tests use --release eg cargo test --workspace --release
- For extra debugs use debug-logs eg --features paper-exact,debug-logs

## Formal Lean Subproject (`formal/superneo-lean`)
- Use this 3-layer layout for each formalized component:
  - Human spec: `formal/superneo-lean/specs/<Name>.spec.md`
  - Typed Lean interface: `formal/superneo-lean/SuperNeo/<Name>Interface.lean`
  - Lean implementation: `formal/superneo-lean/SuperNeo/<Name>.lean`
- Lean build discipline:
  - During iteration, build only the target module(s) you changed and their dependencies, not the whole package.
  - Prefer narrow commands such as `lake build SuperNeo.<Name>` while working.
  - If several Lean modules changed, build the narrowest affected theorem-facing targets that cover those edits.
  - Only once the Lean work is complete, run a full `lake build` to catch package-wide breakage before finishing.
- Closure standard (mandatory): **Paper-faithful proof-complete**.
  - A module is only considered complete when the exact mathematical construction/claim from
    `./formal/superneo-lean/SuperNeo.pdf.md` is proved in Lean at quantified theorem level.
  - Regression checks (`lake exe check`, generated vectors, booleans) are required but are never
    sufficient evidence for completion.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LFDT-Nightstream/Nightstream](https://github.com/LFDT-Nightstream/Nightstream) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
