---
trigger: always_on
description: generates info messages with replacement tactics — prefer the suggested
---

# lean-zip

Lean 4 compression library: zlib via C FFI, plus pure-Lean tar and ZIP archives.
Toolchain: see `lean-toolchain`. Build system: Lake.

## Build and Test

    lake build          # build library + test executable
    lake exe test       # run all tests

Run from the project root. Tests require `testdata/` directory.

**Quality metric**: sorry count — `grep -rc sorry Zip/ || true`

### NixOS / nix-shell

On NixOS, the project's `shell.nix` provides zlib/pkg-config.
If direnv is set up, the environment activates automatically on `cd`.
Otherwise, prefix commands with `nix-shell --run`:

    nix-shell --run "lake build && lake exe test"

**Important:** Lake caches `run_io` results (like `moreLinkArgs`) in
`.lake/`. If you switch between nix-shell and bare shell, or the nix
environment changes, you may need `rm -rf .lake` before building — a
plain `lake clean` is not sufficient to clear the cached link flags.

On systems where zlib is available system-wide (e.g. Ubuntu
with `libz-dev`), no nix-shell wrapper is needed.

## Code Organization

### Source layout

Survey `Zip/` and `ZipTest/` directly. Every source file has a
module-level `/-! ... -/` docstring describing its purpose. Run `ls Zip/**/*.lean`
to orient. Key directories:
- `Zip/` — FFI wrappers and pure-Lean implementations
- `Zip/Native/` — Native Lean implementations (no FFI)
- `Zip/Spec/` — Formal specifications and correctness proofs
- `ZipTest/` — Per-module tests
- Shared utilities (Binary, Handle, BitReader, ZipForStd) live in
  [lean-zip-common](https://github.com/kim-em/lean-zip-common)

### Key documents
    PLAN.md              — Phased roadmap and development cycle (do not modify)
    PROGRESS.md          — Global milestones (updated by summarize agents)
    progress/            — Per-session progress entries (one file per session)
    plans/               — Per-session plan files (one file per active session)
    coordination         — Multi-agent coordination script (uses gh CLI)
    .claude/skills/      — Project-local skill files (proof patterns)

## Quality Standards

### Iterate in place on verified code
Reaching a verified implementation is the *start*, not the finish: a primary
goal of this project is to then make it faster or tighter **while the proof
obligation holds at every commit** — proving the optimized version equal to the
simple one (generational refinement). Proof churn from this is expected and
welcome. Do **not** route a performance change *around* the proofs to avoid
touching them, and do not treat "but it perturbs delicate proofs" as a reason
not to try. The proofs are a ratchet that makes aggressive iteration safe, not
a fragile artifact to protect. When a measured speedup is in tension with proof
churn, default to doing the work. (A risk-framed second opinion will tend to
argue *against* this — weight it accordingly.)

### Development cycle (from PLAN.md)
1. Type signature with `:= sorry`
2. Specification theorems with `:= by sorry`
3. Implementation
4. Auto-solve pass: run `try?` on each `sorry`. If `try?` succeeds, it
   generates info messages with replacement tactics — prefer the suggested
   replacement, but if it looks brittle (e.g. depends on nonlocal simp
   lemmas), use a simpler alternative and note why. Never leave `try?` in
   committed code. Use `bv_decide` when the goal involves `BitVec`.
5. Conformance tests (native vs FFI)
6. Manual proofs for goals that resist automation

### Native implementations
- Place in `Zip/Native/` (e.g. `Zip/Native/Crc32.lean`)
- Formal specs in `Zip/Spec/` (e.g. `Zip/Spec/Crc32.lean`)
- Keep FFI implementations intact as the fast path
- Start simple, optimize later with equivalence proofs

### Specifications

There are three levels of specification quality. Know which you're
writing and be honest about it:

1. **Tautological**: restates the implementation (`f x = fImpl x`).
   Proves nothing useful. Avoid.
2. **Characterizing properties**: mathematical properties independent
   of how the function is computed — algebraic identities
   (e.g. `crc32 (a ++ b) = combine (crc32 a) (crc32 b)`), structural
   invariants (prefix-freeness, Kraft inequality), or invertibility
   theorems (e.g. `decode (encode x) = x`). This is the gold standard.
3. **Algorithmic correspondence**: two implementations of the same
   algorithm agree (e.g. native decoder = reference decoder). Useful
   when the algorithm IS the spec (e.g. RFC pseudocode), but be
   explicit that this is translation validation — it catches bugs in
   the translation between data structures, not logical errors shared
   by both implementations.

When a function's "specification" is an algorithm (RFC pseudocode),
transcribing it into proof-friendly style and proving correspondence
IS the right approach. But characterize the mathematical building
blocks independently where possible. Don't pretend algorithmic
correspondence is characterization. See PLAN.md Phases B3–B4
for how this applies to DEFLATE.

- For optimized versions, specs are equivalence with the simple version.
- Do NOT modify theorem statements just to make proofs easier. If a spec
  is genuinely wrong or too strong, it can be changed — but document the
  rationale in PLAN.md

### Proofs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kim-em/lean-zip](https://github.com/kim-em/lean-zip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
