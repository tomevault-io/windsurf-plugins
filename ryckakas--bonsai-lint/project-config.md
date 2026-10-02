---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`bonsai-lint` is a multi-language cognitive complexity linter written in Rust. It reads source
through tree-sitter grammars without executing any of it; the README lists the supported
languages. One static binary, no language runtime or toolchain required to run the analysis. It
ships as a Cargo workspace plus an independent VS Code extension.

## Commands

```bash
cargo fmt --all -- --check                          # formatting (rustfmt.toml: 100 cols)
cargo clippy --all-targets --all-features -- -D warnings
cargo test --all-features                            # whole workspace
cargo test -p bonsai-lang-php                         # one crate
cargo test -p bonsai-lang-php --test grammar          # one integration test file
cargo build --release                                 # produces target/release/bonsai-lint
RUSTDOCFLAGS="-D warnings" cargo doc --no-deps --workspace --all-features
cargo deny check                                      # advisories, licenses, duplicates: deny.toml
cargo shear                                           # dependencies nothing uses
cargo hack build -p bonsai-lint --feature-powerset    # every language subset, including none
typos                                                 # spelling; deliberate misspellings: _typos.toml
taplo fmt --check                                     # TOML formatting: .taplo.toml
zizmor .github                                        # workflow security: .github/zizmor.yml
cargo llvm-cov --workspace --all-features --summary-only   # test coverage, per file
```

The last seven are separate tools:
`cargo install --locked cargo-deny cargo-shear cargo-hack typos-cli taplo-cli zizmor cargo-llvm-cov`,
and coverage also needs `rustup component add llvm-tools-preview`.

Every action in the workflows is pinned to a commit, with its version in a comment, and every
workflow but `release.yml` defaults to a read-only token. A new `uses:` line follows the same
form, and `zizmor` fails CI until it does.

CI (`.github/workflows/ci.yml`) runs exactly the commands above but the coverage, plus a test
matrix across Linux/macOS/Windows, a build-only check on the MSRV read from `Cargo.toml`
(`rust-version`), and the tests on `x86_64-unknown-linux-musl`, the only build that swaps in
mimalloc (it needs `musl-tools`, so on macOS run it in an `ubuntu` container). Match these locally
before pushing rather than relying on CI to catch it.

Coverage runs in `.github/workflows/coverage.yml`, on pull requests only, and reports rather than
gates. Each run rewrites one comment on the pull request, found by its
`<!-- bonsai-lint-coverage -->` marker, with the totals and a line per crate. The only job allowed
to comment runs none of the pull request's code; a fork's pull request gets the table in the run
summary instead. Doc tests are not counted, since collecting them needs nightly, and neither is the
separate wasm workspace. Mutation testing is too slow for CI and runs on demand; see Testing
conventions.

Every language is behind a cargo feature; `vue` implies `ts` because it reuses that spec. The
registry must keep compiling with any feature subset, including none — this is exercised, not
incidental, so don't add code that assumes a language is always present without a `#[cfg]` guard
matching the existing pattern.

VS Code extension (`editors/vscode/`, own `package.json`, independent version number):

```bash
cd editors/vscode
npm ci
npx tsc -p . --noEmit      # typecheck
npm test                    # compiles then runs node --test on out/*.test.js
npx --yes @vscode/vsce package --out /tmp/extension.vsix
```

PyPI wheels and the pre-commit hook (`pypi/`, stdlib-only Python 3.11+, which macOS's `python3`
is not, so `uv` supplies it). CI runs these as its `PyPI wheel builder` and `pre-commit hook`
jobs:

```bash
uv run --no-project --python 3.12 python -m unittest discover pypi   # the wheel builder's tests
uv run --no-project --python 3.12 python pypi/build_wheels.py binary \
    --binary target/debug/bonsai-lint --target aarch64-apple-darwin --out /tmp/wheels
bash pypi/hook-smoke.sh /tmp/wheels   # pre-commit try-repo on the committed hook, from that wheel
```

## Architecture

```
crates/
├── bonsai-core/        the scorer: parsed tree in, scores out. No I/O, no serde, no grammars.
├── bonsai-lang-go/     Go node kinds, field names and hooks, and its generated-file check
├── bonsai-lang-java/   Java node kinds, field names and hooks, and its generated-file check
├── bonsai-lang-php/    PHP node kinds, field names and hooks
├── bonsai-lang-python/ Python node kinds, field names and hooks, and its generated-file check
├── bonsai-lang-ts/     TypeScript and TSX, sharing one spec across both dialects
├── bonsai-lang-vue/    Vue SFCs: locates the script blocks, scores them with the TS spec
├── bonsai-engine/      registry, configuration, domains, baselines, the scan driver
├── bonsai-lint/        the CLI, producing the `bonsai-lint` binary
├── bonsai-testkit/     the grammar contract harness, used by every language crate

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ryckakas/bonsai-lint](https://github.com/ryckakas/bonsai-lint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
