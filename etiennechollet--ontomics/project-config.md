---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## ABSOLUTE RULE — No AI attribution

**NEVER mention Claude Code, Anthropic, ChatGPT, Codex, OpenAI, or any AI tool in commits, PRs, issues, comments, or any git artifact.** No "Co-Authored-By", no "Generated with", no AI attribution of any kind. This applies to all text that enters the git history or appears on GitHub.

## ABSOLUTE RULE — Worktree workflow

**ALL source code and test changes MUST go through a worktree branch and PR.** Never commit implementation work directly to main. **NEVER edit a single file before the worktree exists.**

**The worktree MUST be created BEFORE any files are edited. This is the very first action for any implementation task. No exceptions. Do not edit on main and stash/move retroactively.**

Worktree location: `./worktrees/per-<issue-number>-<issue-name>` (e.g., `worktrees/per-42-testbed-thread-cap`). **NEVER place worktrees outside the project root (e.g., `../`).**

Workflow:
1. Create a feature branch: `epc28/<description>` (associate with a Linear issue when one exists)
2. Create the worktree: `git worktree add -b epc28/<name> worktrees/per-<issue>-<name> main`
3. Work ONLY inside that worktree — all edits, tests, and commits happen there
4. Create a PR from the feature branch to main
5. Merge to main only after review/validation

Direct commits to main are only acceptable for version bumps and release automation (cargo-release).

## ABSOLUTE RULE — Embeddings must stay enabled

**NEVER disable embeddings** (`embeddings.enabled = false`) in any `.ontomics/config.toml` for any codebase without explicit permission from the user. Embeddings are a core part of the pipeline — disabling them silently degrades concept discovery, clustering, and suggest_name quality.

## ABSOLUTE RULE — Tests are sacred

**NEVER modify, delete, weaken, skip, or rewrite any test without explicit permission from the user.** This is the single most important rule in this project. No exceptions.

- If a test fails, the implementation is wrong — fix the implementation.
- Do not change assertions, expected values, or test structure to make a failing test pass.
- Do not add `#[ignore]`, comment out, or remove failing tests.
- Do not refactor tests "for clarity" or "consistency" unless explicitly asked.
- The only time a test may be modified is when the user explicitly says to modify that specific test.
- Do not widen assertion windows (e.g., top_n thresholds) to absorb implementation regressions.
- Do not remove expected entities, concepts, or conventions from test expectations.
- Do not replace specific assertions with weaker structural checks.
- Do not "update expectations to match reality" — reality must match expectations.
- Do not relax naming checks, convention checks, or entity lists because the tool produces wrong output.
- Do not move must-contain items to should-contain or drop them entirely.
- Failing testbed tests are bugs in `src/`, not bugs in `tests/`.
- The testbed expectations are the definition of done — they define what ontomics MUST produce.
- Any change to analysis logic (TF-IDF, stop words, convention detection) that breaks testbed expectations must be fixed in the analysis code until the tests pass, not by weakening the tests.
- When in doubt: the test is right, the implementation is wrong.

## Terminology

- **"pi"** — Refers to the [pi agent harness](https://github.com/badlogic/pi-mono), NOT Raspberry Pi or the number. When the user mentions "pi", they are talking about this agent harness UI.

## What this is

ontomics is a Rust MCP server that extracts domain ontologies from codebases (Python, TypeScript, JavaScript, Rust). It parses source files with tree-sitter, clusters related identifiers by embedding similarity, detects naming conventions, clusters functions by behavioral similarity using code embeddings, and exposes the results as MCP tools. Runs locally with no API keys.

## Build and test

```bash
cargo build --release          # production binary
cargo build                    # debug build
cargo test                     # all tests (inline in each module)
cargo test --lib config        # tests in a single module
cargo clippy                   # lint (must pass with zero warnings)
```

Optional feature flag: `cargo build --features lsp` enables pyright-based LSP enrichment for inheritance chains.

## Releases

**All four version sources MUST be bumped in lockstep on every release:**

1. `Cargo.toml` — `version` field
2. `Cargo.lock` — auto-updated via `cargo update -p ontomics`
3. `server.json` — both top-level `version` and `packages[0].version`

**Release procedure:**
1. Bump versions in `Cargo.toml` and `server.json` (all three locations)
2. Run `cargo update -p ontomics` to sync `Cargo.lock`
3. Commit: `chore: Release ontomics version X.Y.Z`
4. Tag: `git tag vX.Y.Z`
5. Push commit and tag: `git push && git push origin vX.Y.Z`

CI (`release.yml`) handles everything from there: builds binaries, publishes to GitHub Releases, npm, Homebrew, and MCP Registry. All four channels must show the same version after a release.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EtienneChollet/ontomics](https://github.com/EtienneChollet/ontomics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
