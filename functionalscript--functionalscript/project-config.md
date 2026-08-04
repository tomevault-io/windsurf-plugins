---
trigger: always_on
description: This repository contains both Node.js (TypeScript) and Rust code. Check the `todo/` directories for existing issues.
---

# AGENT Instructions

This repository contains both Node.js (TypeScript) and Rust code. Check the `todo/` directories for existing issues.

## Requirements

### Container

1. **Node.js 22 or later** should be installed.
2. **FunctionalScript** should be installed. `npm install -g functionalscript`.
3. **Rust** should be installed.

### Dependencies

- Install Node dependencies with `npm ci`.
- Install Rust dependencies with `cargo fetch`.

## Update

It's recommended to run `npm run update` after changing the source code.

## Testing

- Run `npx tsc` to type-check using the repository's version of TypeScript.
- Run `fjs t` to test FunctionalScript (`.f.ts`) files with Node 22+.
- Run `cargo test` to test the Rust crate in `nanvm-lib`.
- Run `cargo clippy` to lint the Rust crate.
- Run `cargo fmt -- --check` to verify formatting.
- To run only the tests under a specific directory, `cd` into it and run `npm run fst`. This scans for `test.f.ts` files in that subtree and reports per-test results.
- New FunctionalScript (`.f.ts`) modules and functions must have **100% proof coverage** across every dimension: every exported function called, every line executed, and every branch (both sides of each conditional) taken. A new `module.f.ts` ships with a co-located `proof.f.ts` (its `proof` export) that exercises all of its exports along all code paths — partial coverage of new code is not acceptable. If a line or branch genuinely cannot be reached, restructure the code so it isn't there rather than leaving it uncovered.
- Assert results in `proof` code with `assert`/`assertEq` from `fs/asserts/module.f.ts`, not a hand-written `if (cond) { throw ... }`. A local `if/throw` in a test is itself a new branch for the coverage tool to track, and its failure side is normally never exercised (the test is expected to pass), so it lands as a permanently-uncovered branch in the very module meant to close coverage gaps. `assert`/`assertEq` push that branch into a shared helper whose own branches are already fully covered elsewhere, so the call site adds no new uncovered branch.
- Never use `try`/`catch` in `.f.ts` files — FunctionalScript itself has no `try`/`catch` and isn't planning to add it soon. To test that a call throws, nest the test function under a `throw` property key instead of wrapping it in `try`/`catch` (see `fs/asserts/proof.f.ts`). The test runner (`fs/emergent_testing/module.f.ts`) treats `throw` as a structural marker: any function reachable under a `throw` key gets `throws: true`, and the runner inverts the sandboxed result so a thrown error counts as a pass — with no manual `caught`/`threw` flag or `assert` needed. Treat `throw` in FunctionalScript as a panic (like Rust's `panic!`, Go's `panic`, or Java/C#'s unchecked `RuntimeException`), not as a language-level `Result`/checked-exception value: nothing in FunctionalScript can catch it, so a thrown payload is never pattern-matched or branched on by other FunctionalScript code, and a correctly working program should never throw at all. Recoverable failure belongs in `Result` (`fs/types/result`), which callers actually destructure and is worth asserting on precisely; a `throw`'s payload is read only by a human or external tooling after something has already gone wrong, so don't over-invest proof effort in checking its exact value — whether it threw is normally the part of the contract that matters.

## Documentation

Use JSDoc for documenting TypeScript files. Every module should start with this header:

```ts
/**
 * <...Module documentation...>
 *
 * @module
 */
```

where `<...Module documentation...>` should be documentation for the module.

## Design and Implementation

- **Always prefer simplicity and quality over optimization.** Never optimize prematurely, and especially never at the cost of simplicity. A simple, correct, generic solution comes first; optimization work starts only after confirming it is actually needed (a measured problem or a real limit being hit, not a hunch), and even then it is a **separate task**: file it as its own `todo/` issue instead of folding it into the current change. When that task is taken up, still solve the problem in a generic way — improve the algorithm, the data structure, or the API — instead of hacking special cases into an otherwise general design (byte-prefix sniffing instead of real parsing, key-order assumptions, hardcoded fast paths). A documented implementation limit that a later generic improvement can lift (e.g. a size bound on a buffering parser) is an acceptable interim answer; a semantic assumption baked into a format or contract for speed is not.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [functionalscript/functionalscript](https://github.com/functionalscript/functionalscript) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
