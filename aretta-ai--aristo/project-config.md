---
trigger: always_on
description: **This file is law.** Every Claude Code session in this repo MUST read this file before touching code, and MUST obey every rule below. These rules are not aspirational — they are how we work. Violating any of them is grounds for the user to revert your change.
---

# CLAUDE.md — Aristo working agreement

**This file is law.** Every Claude Code session in this repo MUST read this file before touching code, and MUST obey every rule below. These rules are not aspirational — they are how we work. Violating any of them is grounds for the user to revert your change.

If you find yourself rationalizing an exception ("just this once," "the rule doesn't really apply here"), STOP. The rule applies. If the rule is genuinely wrong, surface that to the user and propose a CLAUDE.md edit — do not silently bypass it.

---

## §1. Commit size — small or medium ONLY

- **Large commits are FORBIDDEN.** No exceptions.
- Heuristic: if the diff exceeds ~200 changed lines OR touches more than ~5 files for unrelated reasons, SPLIT IT.
- One logical change per commit. If the message needs the word "and," it is two commits.
- The only allowed "wide" commit is a mechanical, atomic refactor (e.g. a project-wide rename) that is trivially reviewable as a single operation. Surface these to the user before making them.

## §2. Commit messages — semantic / conventional

Required prefix from this exact set:

| Prefix | Use for |
|---|---|
| `feat:` | new user-visible functionality |
| `fix:` | bug fix |
| `refactor:` | code change that neither fixes a bug nor adds a feature |
| `perf:` | performance improvement, no behavior change |
| `docs:` | documentation only (including this file, README, CHANGELOG-only edits — but see §3) |
| `test:` | tests only |
| `build:` | build system, dependencies, `Cargo.toml`, workspace config |
| `chore:` | housekeeping that doesn't fit above |
| `ci:` | CI / GitHub Actions config |

Optional scope in parens: `feat(macros): ...`, `fix(cli): ...`, `build(workspace): ...`.

**Banned messages:** `wip`, `stuff`, `updates`, `misc`, `fixes`, `progress`, `more changes`. Say what changed.

## §3. CHANGELOG.md — one line per commit, in the same commit

- **Every commit MUST add at least one bullet** to the `## [Unreleased]` section of `CHANGELOG.md`, describing what changed in customer-facing language.
- The CHANGELOG bullet ships **in the same commit** as the code change. Never a separate "update changelog" commit.
- Format: `- <area>: <what changed and why a user cares>`. Examples:
  - `- macros: \`#[aristo::intent]\` now accepts multi-line text without escaping.`
  - `- cli: \`aristo verify --audit\` exits non-zero on stale/refuted proofs for CI gating.`
- At release: promote `## [Unreleased]` to `## [vX.Y.Z] — YYYY-MM-DD`. The `[Unreleased]` block must read coherently as a release-note draft when scanned end-to-end.

## §4. Test-first — no test, no claim of correctness

- **Write the test BEFORE the implementation**, as far as possible. Goal: surface ambiguity. If you cannot write the test, you do not yet know what you are building — go clarify before writing code.
- **NO TEST = NO CLAIM OF CORRECTNESS.** "Should work," "looks right," "compiles," "I checked it manually" are all NOT correctness. The bar is: a test demonstrates the behavior, the test passes, the test is committed alongside the code.
- The TDD inner loop is local; what gets committed is always green:
  1. Write failing test → run it → confirm it fails for the right reason.
  2. Write implementation → run test → it passes.
  3. Run the full check suite (§6).
  4. Commit (test + impl + CHANGELOG bullet, all together).
- **Scenario-level extension** (per §12A): at the start of every slice, the spec scenarios that define the slice's success criterion get promoted from `_pending/`|`_blocked/` to `active/` BEFORE any impl is written. Those scenarios are the slice's red tests at the spec level; unit tests are the slice's red tests at the function level. Both feed §4's "red → green" loop.

## §5. Autonomous diagnosis when coverage is good

- When the area you are touching has good test coverage, **diagnose and fix problems autonomously**. Read the failure, form a hypothesis, test it, iterate. Do not punt to the human after a single failure — that is what the test suite is for.
- When coverage is thin and behavior is ambiguous, **stop and surface the ambiguity** to the user before guessing. Better to ask than to encode the wrong invariant.

## §6. Every commit passes ALL checks

Before `git commit`, you MUST run and pass:

```sh
cargo fmt --all -- --check
cargo check --workspace --all-targets
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```

- All four MUST be green. If clippy fires a warning, fix the root cause — do NOT add `#[allow(...)]` without first asking the user whether the lint should be suppressed for this case.
- If a pre-commit hook fails, FIX THE ROOT CAUSE. Never use `--no-verify`. Never use `--no-gpg-sign` unless the user explicitly asks for it.
- A failed pre-commit hook means the commit DID NOT happen. After fixing, create a NEW commit — never `--amend`, because `--amend` would modify the previous commit.

## §7. No hacky fixes — refactor before patching

- Architect for **maximal code reuse**. If you find yourself copy-pasting, factor.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aretta-ai/aristo](https://github.com/aretta-ai/aristo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
