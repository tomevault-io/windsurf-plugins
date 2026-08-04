---
trigger: always_on
description: Guidance for AI coding agents working in this repo.
---

# AGENTS.md

Guidance for AI coding agents working in this repo.

## What this is

Rust rewrite of [dynamik-dev/bully](https://github.com/dynamik-dev/bully) — local CI for AI coding agents. Status: **0.4 "checks pipeline" + self-trust bash gate all merged; `ironlint verify` and the full `doctor` expansion remain pending.** A check is `files` (globs) + `run` (or `steps`) + `on` (lifecycle); ironlint matches a touched file to checks, runs each command with the ABI on env + proposed content on stdin, and reads only the exit code — **any nonzero exit (1–125) blocks**. No per-rule engines, no severity, no LLM. CLI ships `check` (with `--file`, `--diff`, or bare for a repo-wide sweep), `validate`, `init` (scaffolds `.ironlint.yml` AND onboards ironlint's hook into detected coding agents — claude-code, codex, pi, opencode), `explain`, `show-resolved-config`, `doctor` (reports per-harness adapter status in the `checks[]` array), `trust` (blesses the out-of-repo store; `check` fails closed — exit 4 — on untrusted config/checks), `gate-bash` (pure-Rust Bash-command classifier the adapters' Bash branches shell out to — blocks `ironlint trust` and Bash writes to the policy surface; not a `check`, not trust-gated, runs with no `.ironlint.yml`), `update` (self-updates the binary to the latest GitHub release via the dist install receipt), `watch` (a read-only live TUI over the `.ironlint/log.jsonl` telemetry log), and `schema` (prints the canonical check-authoring guide — the same content `ironlint init` installs into each agent as the `ironlint-config` skill). **Every adapter also gates the agent's Bash tool** via `gate-bash`, closing the self-trust escape hatch (an agent can't run `ironlint trust` or write `.ironlint.yml` / `.ironlint/scripts/` through Bash). Authoritative design: `specs/2026-06-28-ironlint-checks-pipeline-design.md`; bash-gate design: `docs/superpowers/specs/2026-07-06-bash-gate-self-trust-prevention-design.md`; per-phase plans in `plans/`.

**Not yet built (later plans):** `ironlint verify` + the full `doctor` expansion.

## Rules

- Bug fixes start with a failing test (use the test-writing skill). The failing test becomes regression coverage.
- After completing a coding task, request code review from a separate agent.
- Your code reviews are reviewed by the principal engineer — do deep work.
- Tool hasn't shipped; no hedging.
- Rust files under `crates/*/src/` must meet ≥80% **region** coverage (distinct decision points — branches, short-circuits, match arms — not executed lines). CI enforces per-file via `scripts/ci-coverage.sh` (cargo-llvm-cov). Code added without bringing the file to the gate breaks the build.
- Cognitive complexity per function is capped at **15** via clippy (`clippy.toml`, with `#![warn(clippy::cognitive_complexity)]` at each crate root). Refactor over annotate; reach for `#[allow(clippy::cognitive_complexity)]` only when complexity is intrinsic to the function and decomposing would scatter the flow — document why.
- Mutation testing is a **local, ad-hoc** investigative tool, not a CI gate (would burn runner minutes). `cargo install cargo-mutants` once, then point it at a file or diff: `cargo mutants --file 'crates/ironlint-core/src/<name>.rs'` for one file, or `git diff main.. > pr.diff && cargo mutants --in-diff pr.diff` for the PR. A surviving mutant means tests executed the code but didn't verify what it does — treat survivors in code you touched as a coverage gap.
- Clean up build artifacts you produced once the task is done. If you ran `cargo build --release` or built a one-off binary to verify behavior, drop it with `cargo clean -p <crate>` or `rm target/release/<bin>` after verification. Same for throwaway files like `pr.diff`, ad-hoc tarballs, scratch `cargo mutants` output, or any binary built for a single check. The persistent `target/` you're actively iterating in stays — this rule is about artifacts *this task* created, not the working tree.

## Commands

```bash
cargo build --release                       # produces ./target/release/ironlint
./target/release/ironlint check              # bare = repo-wide sweep (batched where checks allow)
cargo test                                  # all workspace tests
cargo test -p ironlint-core                 # core only
cargo test -p ironlint-cli                  # CLI only
cargo test --test cli_e2e_gates             # single integration test file (checks pipeline)
cargo test <name>                           # filter by test-fn name
cargo clippy --all-targets -- -D warnings   # lint
cargo fmt
bash scripts/ci-coverage.sh                 # per-file ≥80% region-coverage gate (matches CI)
```

CLI tests use `assert_cmd` against the compiled binary. (`insta` snapshots may exist for some surfaces — `cargo insta review` after an intentional shape change.)

## Architecture

Cargo workspace, three crates:

- **`ironlint-core`** — library. Modules:
  - `config` — parse the checks YAML (`Config { extends, execution, checks }`, `Check { files, run, steps, on, name }`, `Step { name, run }`), glob scope matching (`scope.rs`), `extends:` resolution (`extends.rs`)
  - `diff` — unified-diff parser (used by CLI `--diff` to enumerate changed files)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ironlint/ironlint](https://github.com/ironlint/ironlint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
