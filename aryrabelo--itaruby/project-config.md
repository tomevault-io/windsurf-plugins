---
trigger: always_on
description: - DOX is a self-documenting AGENTS.md hierarchy installed here (OMP-focused).
---

# DOX framework — itaruby

- DOX is a self-documenting AGENTS.md hierarchy installed here (OMP-focused).
- Every agent must follow DOX instructions across any edits.

## Project

itá means "stone" in Tupi. itaruby looks at the stone and says whether it is a
real ruby: a Ruby type checker written in Rust, inference-first (in the style of
ty/Pyrefly, not Sorbet), with a real-time LSP server. Workspace of 4 crates
(`crates/itaruby_syntax` — parsing via ruby-prism; `crates/itaruby_semantic` —
index/types/checking as salsa queries; `crates/itaruby_server` — LSP;
`crates/itaruby` — the `ita` binary with `check` and `server`). Entry points:
`crates/itaruby/src/main.rs` (CLI) and `crates/itaruby_server/src/main_loop.rs`
(LSP).

**The project's bar: itaruby is the typechecker add-on for ruby-lsp, done
right.** The Shopify ruby-lsp has no type system (its own docs point users to
Sorbet/Steep) and its roadmap asks for "Allow the Ruby LSP to connect to a
typechecker add-on" — an aspiration, not an implementation. The gem in
`scripts/ruby-lsp-itaruby/` is that add-on; the VS Code extension in
`scripts/vscode/` is a reference client, never the main channel. Sorbet is the
precision adversary: itaruby is benchmarked against it on public Rails apps
(rails/rails, mastodon, discourse, gitlab-foss) on both speed and true errors
found, and the launch bar is winning on both.

## Core Contract

- AGENTS.md files are binding work contracts for their subtrees.
- Every meaningful change requires a DOX pass before the task is done: update
  the closest owning AGENTS.md when a change affects purpose, scope, ownership,
  contracts, workflows, constraints, or this index. Remove stale text
  immediately. Small no-behavior edits may leave docs unchanged — the pass
  still happens.
- Rules live in exactly one owning file. Child docs never restate parent
  rules. If two rules conflict, fix the docs in the same change —
  contradictions are bugs.
- Lessons learned in-session become rules with a date and a marker:
  `(learned YYYY-MM-DD, binding)`.

## Global Contracts

- **Invariant #1 (inviolable):** `Ty::Unknown` NEVER produces a diagnostic. A
  false negative is acceptable; ONE false positive on the benchmark corpora is
  a failure. Navigation follows the same rule: no-answer is acceptable, a
  wrong answer never is.
- **Fail-closed everywhere:** where itaruby cannot prove, it stays silent and
  the gap becomes roadmap — never a diagnostic (binding, 2026-08-24).
- Conflicting superclass headers are unknown ancestry, not a file-order
  winner (learned 2026-09-22, binding). Compare resolved identities in each
  header's lexical scope; same-base and bare reopenings remain valid.
  Constants stay suppression-only under conflict, including global-fallback
  collisions and intermediate qualified prefixes. Remove conflict-dependent
  headers before schema/RBI consumers inspect their spelling. Every alias RHS
  uses its write-site scope; known lexical aliases precede global barriers,
  and cycles/hop limits are inconclusive, not absent constants. Focused proof:
  `conflicting_superclasses`, `ancestry_review_controls`, and
  `scripts/conflicting-superclasses-mutants.py` (gate c1 and CI); public
  baselines require a fresh measured audit after source changes.
- A code path that turns silence into an error must never read absence of
  evidence as evidence of absence (learned 2026-08-24, binding): a gate that
  finds no signal on an upward search (e.g. no Gemfile discovered within N
  levels) fails closed to "couldn't tell, treat as present" rather than
  "confirmed absent, diagnose" — an unresolvable result is never license to
  accuse.
- The private benchmark corpora (corpus-a, corpus-b, corpus-c) are sacred:
  their expected error sets are recorded in `scripts/corpus-baseline.txt` as
  SHA-256 hashes only — never a path, class, method, table, or column name in
  clear text. Any new diagnostic on a corpus must be proven a true positive by
  reading the flagged code, or the change is reverted. Ratchets turn false
  positives into contracts: a wrong diagnostic that enters a baseline stays
  green forever (learned 2026-08-20, binding).
- The corpora are company code and each lives on exactly one machine — never
  rsync/copy a corpus between machines. Every corpus is READ-ONLY, always.
  Their real paths live only in the gitignored `scripts/corpora-local.txt` of
  each machine that hosts them.
- The secrecy wall above covers five surfaces, all audited: versioned file
  content; commit messages; PR titles and bodies; PR/issue comments and
  reviews; and tool-generated artifacts that get versioned (learned
  2026-08-20, binding). Names that only exist in a client's code never enter
  this repository; public gem API names are always fine.
- software-factory is alive (`sf check` in the pre-commit hook). NEVER loosen
  `.software-factory/policy.yaml` or the ratchet — a rule that blocks you is
  a finding about your change, not about the rule. That sentence used to be
  the only thing enforcing it; since 2026-08-26 `L2.POLICY_ONLY_TIGHTENS`
  and `L2.FACTORY_CONFIG_IS_LOCKED` make a loosening fail the build instead
  of relying on a reader (learned 2026-08-26, binding: a rule that lives

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aryrabelo/itaruby](https://github.com/aryrabelo/itaruby) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
