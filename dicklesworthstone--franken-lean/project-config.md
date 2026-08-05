---
trigger: always_on
description: > Guidelines for AI coding agents working in this Rust codebase.
---

# AGENTS.md — franken_lean

> Guidelines for AI coding agents working in this Rust codebase.

---

## RULE 0 — THE FUNDAMENTAL OVERRIDE PREROGATIVE

If I tell you to do something, even if it goes against what follows below, YOU MUST LISTEN TO ME. I AM IN CHARGE, NOT YOU.

---

## RULE NUMBER 1: NO FILE DELETION

**YOU ARE NEVER ALLOWED TO DELETE A FILE WITHOUT EXPRESS PERMISSION.** Even a new file that you yourself created, such as a test code file. You have a horrible track record of deleting critically important files or otherwise throwing away tons of expensive work. As a result, you have permanently lost any and all rights to determine that a file or folder should be deleted.

**YOU MUST ALWAYS ASK AND RECEIVE CLEAR, WRITTEN PERMISSION BEFORE EVER DELETING A FILE OR FOLDER OF ANY KIND.**

---

## Irreversible Git & Filesystem Actions — DO NOT EVER BREAK GLASS

1. **Absolutely forbidden commands:** `git reset --hard`, `git clean -fd`, `rm -rf`, or any command that can delete or overwrite code/data must never be run unless the user explicitly provides the exact command and states, in the same message, that they understand and want the irreversible consequences.
2. **No guessing:** If there is any uncertainty about what a command might delete or overwrite, stop immediately and ask the user for specific approval. "I think it's safe" is never acceptable.
3. **Safer alternatives first:** When cleanup or rollbacks are needed, request permission to use non-destructive options (`git status`, `git diff`, `git stash`, copying to backups) before ever considering a destructive command.
4. **Mandatory explicit plan:** Even after explicit user authorization, restate the command verbatim, list exactly what will be affected, and wait for a confirmation that your understanding is correct. Only then may you execute it.
5. **Document the confirmation:** When running any approved destructive command, record (in the session notes / final response) the exact user text that authorized it, the command actually run, and the execution time.

---

## Branch Policy

- Primary branch is `main`.
- Do not reference `master` in docs/scripts.
- If release instructions require sync, push `main:master` after `main`.

---

## Project Mission

`franken_lean` (**FrankenLean**, crate prefix `fln-`) is a **ground-up, native-Rust reimplementation of the entire Lean 4 toolchain** — parser, macro engine, elaborator, unifier, instance engine, tactic framework, simp and the decision procedures, trusted kernel, compiler, VM, runtime/ABI twin, module codec, build system, and language server — that is a **drop-in replacement at the binary surfaces**: the source language, the `.olean` object format (read *and* write), the `lean_object` C ABI (`lean.h` twin), the LSP wire protocol with the `$/lean/*` extensions, and the `lean`/`leanc`/`lake` CLI surfaces. Under those familiar surfaces it is deliberately better where better is sound: deterministic under parallelism, declaration-granular incremental, memory-shared, provenance-transparent.

**The Oracle-Only Law (D8) is constitutional:** no upstream implementation code ever executes as a component of FrankenLean — not the C++ kernel, not the self-hosted `Lean.*` elaborator sources, not stage0. The Reference toolchain (`leanprover/lean4` at the pinned epoch tag) appears in exactly one place: inside the **Tribunal**, as the differential oracle, fixture generator, and census-extraction source. The only Lean code FrankenLean ever *executes* is user code (mathlib's tactics, downstream libraries, lakefiles, `#eval`) on our own VM (Golem) against a natively-implemented `Lean.*` surface (the Native Mirror).

The leapfrog is not one trick; it is the *composition* of eight bets, each at or beyond the current frontier, made feasible only because the foundation libraries already exist:

- **B1 — The Ledger.** The environment is a Merkle DAG of content-addressed declarations; builds are memoized queries over it; a one-line leaf edit re-elaborates its true dependency cone (seconds), not its file cone (hours); the cloud cache is native CAS sync over atp.
- **B2 — The Native Mirror.** The entire `Lean.*`/`Init`/`Std` builtin surface is served *natively*: toolchain-API symbols are Rust implementations registered under upstream names behind a census-generated façade; pure library code is upstream-authored *source* elaborated by our own toolchain; user metaprograms run on our VM and cannot tell.
- **B3 — Kernel with receipts.** A ≤ 12 KLOC dual-engine trusted checker (certified small-step + NbE accelerator, cross-checked), deterministic fuel parity, proof-certificate export by default, consensus receipts with an independent in-repo checker plus external witnesses — disagreement halts, never outvotes.
- **B4 — Deterministic parallel elaboration.** Declaration-granular dataflow parallelism with speculative execution and deterministic merge: results are schedule-independent by construction (FL-INV-01), tested at {1, 8, 32} threads on every commit.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/franken_lean](https://github.com/Dicklesworthstone/franken_lean) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
